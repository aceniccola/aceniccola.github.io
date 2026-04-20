```cpp
#include <cuda_runtime.h>
#include <stdio.h>

// Define the tile size. A 32x32 tile matches the size of a CUDA warp (32 threads)
// per dimension, which is highly efficient for hardware execution.
#define TILE_SIZE 32

/**
 * @brief Tiled Matrix Multiplication Kernel (C = A * B)
 * * @param A Pointer to Matrix A in Global Memory
 * @param B Pointer to Matrix B in Global Memory
 * @param C Pointer to output Matrix C in Global Memory
 * @param width The width/height of the square matrices
 */
__global__ void tiled_gemm_kernel(const float* A, const float* B, float* C, int width) {
    
    // -------------------------------------------------------
    // 1. Thread Identification
    // -------------------------------------------------------
    // Which row and column of the final output matrix C is this thread responsible for?
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    int col = blockIdx.x * blockDim.x + threadIdx.x;

    // Allocate Shared Memory (The "Conference Room Whiteboard")
    // These tiles will hold the sub-matrices of A and B for the current step.
    __shared__ float s_A[TILE_SIZE][TILE_SIZE];
    __shared__ float s_B[TILE_SIZE][TILE_SIZE];

    // Local register to accumulate the dot product result for this thread's cell in C
    float c_value = 0.0f;

    // -------------------------------------------------------
    // 2. The Tiling Loop (Sliding across A and down B)
    // -------------------------------------------------------
    // We divide the total width of the matrix by the TILE_SIZE to determine
    // how many "steps" or "chunks" we need to process.
    int num_tiles = width / TILE_SIZE;

    for (int t = 0; t < num_tiles; ++t) {
        
        // --- PHASE A: Collaborative Loading ---
        // Each thread in the block loads exactly ONE element from Global Memory 
        // into Shared Memory. 
        
        // Calculate the global memory indices for the current tile
        // For A: The row stays the same, but we slide horizontally (t * TILE_SIZE)
        int a_idx = row * width + (t * TILE_SIZE + threadIdx.x);
        
        // For B: We slide vertically down (t * TILE_SIZE), but the column stays the same
        int b_idx = (t * TILE_SIZE + threadIdx.y) * width + col;

        // Load into shared memory
        s_A[threadIdx.y][threadIdx.x] = A[a_idx];
        s_B[threadIdx.y][threadIdx.x] = B[b_idx];

        // BARRIER: Wait until ALL threads in the block have finished loading their element.
        // If we don't sync here, some threads might start calculating before the data arrives.
        __syncthreads();


        // --- PHASE B: Compute ---
        // Now that the 32x32 tiles of A and B are safely in ultra-fast Shared Memory,
        // we can perform the dot product for this chunk.
        
        // Notice we are reading from s_A and s_B (Shared Memory), NOT global memory!
        for (int k = 0; k < TILE_SIZE; ++k) {
            c_value += s_A[threadIdx.y][k] * s_B[k][threadIdx.x];
        }

        // BARRIER: Wait until ALL threads have finished computing this tile.
        // If we don't sync here, a fast thread might loop around and start overwriting 
        // s_A or s_B with data for the *next* tile while slower threads are still 
        // trying to read data for the *current* tile.
        __syncthreads();
    }

    // -------------------------------------------------------
    // 3. Write to Output
    // -------------------------------------------------------
    // After sliding across all tiles, the dot product is complete.
    // We do ONE write to slow Global Memory to store the final result.
    C[row * width + col] = c_value;
} ```

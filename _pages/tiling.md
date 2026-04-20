```cpp
#include <cuda_runtime.h>

#define TILE_SIZE 32

/**
 * @brief General Tiled Matrix Multiplication Kernel (C = A * B)
 * @param A Pointer to Matrix A (Size: M x K)
 * @param B Pointer to Matrix B (Size: K x N)
 * @param C Pointer to Output Matrix C (Size: M x N)
 * @param M Number of rows in A and C
 * @param N Number of columns in B and C
 * @param K Inner dimension (Cols of A, Rows of B)
 */
__global__ void general_tiled_gemm_kernel(const float* A, const float* B, float* C, 
                                          int M, int N, int K) {
    
    // 1. Global Coordinates for the Output Matrix C
    int row = blockIdx.y * blockDim.y + threadIdx.y;
    int col = blockIdx.x * blockDim.x + threadIdx.x;

    // Allocate Shared Memory Tiles
    __shared__ float s_A[TILE_SIZE][TILE_SIZE];
    __shared__ float s_B[TILE_SIZE][TILE_SIZE];

    float c_value = 0.0f;

    // 2. Calculate the sliding distance along the inner dimension (K)
    // We use ceil division to ensure we process the "leftover" chunk at the end of K
    int num_tiles = (K + TILE_SIZE - 1) / TILE_SIZE;

    for (int t = 0; t < num_tiles; ++t) {
        
        // --- PHASE A: Load Matrix A with Boundary Checks ---
        // A thread is assigned to load a specific cell from A.
        // The row is fixed, but the column slides horizontally by (t * TILE_SIZE)
        int a_col = t * TILE_SIZE + threadIdx.x;
        
        // Check if the physical thread is within the actual bounds of Matrix A
        if (row < M && a_col < K) {
            // Note how we calculate a 1D index for a 2D matrix: (row * row_width + col)
            s_A[threadIdx.y][threadIdx.x] = A[row * K + a_col];
        } else {
            // ZERO-PADDING: If the thread is outside the matrix, load a 0.
            // This ensures the math later on isn't corrupted by garbage memory.
            s_A[threadIdx.y][threadIdx.x] = 0.0f;
        }


        // --- PHASE B: Load Matrix B with Boundary Checks ---
        // The column is fixed, but the row slides vertically down by (t * TILE_SIZE)
        int b_row = t * TILE_SIZE + threadIdx.y;
        
        if (b_row < K && col < N) {
            s_B[threadIdx.y][threadIdx.x] = B[b_row * N + col];
        } else {
            s_B[threadIdx.y][threadIdx.x] = 0.0f; // ZERO-PADDING
        }

        // Wait for all threads to finish loading and padding
        __syncthreads();


        // --- PHASE C: Compute ---
        // We always loop a full TILE_SIZE times.
        // Because we padded with 0.0f, out-of-bounds elements simply add 0 to c_value!
        for (int i = 0; i < TILE_SIZE; ++i) {
            c_value += s_A[threadIdx.y][i] * s_B[i][threadIdx.x];
        }

        // Wait for all math to finish before overwriting shared memory in the next loop
        __syncthreads();
    }

    // --- PHASE D: Write to Output Matrix C ---
    // Finally, ensure only threads that correspond to valid cells in Matrix C write the result.
    // The "ghost threads" that fell off the edge during the grid launch do nothing here.
    if (row < M && col < N) {
        C[row * N + col] = c_value;
    }
}

#### --- ####
void launch_general_gemm(float* d_A, float* d_B, float* d_C, int M, int N, int K) {
    
    dim3 threadsPerBlock(TILE_SIZE, TILE_SIZE);

    // We must use "ceiling division" to calculate the number of blocks.
    // Integer division rounds down. If N=100 and TILE_SIZE=32, 100/32 = 3.
    // The formula (X + TILE_SIZE - 1) / TILE_SIZE forces the division to round UP.
    
    int blocks_x = (N + TILE_SIZE - 1) / TILE_SIZE; // Covers columns
    int blocks_y = (M + TILE_SIZE - 1) / TILE_SIZE; // Covers rows
    
    dim3 numBlocks(blocks_x, blocks_y);

    general_tiled_gemm_kernel<<<numBlocks, threadsPerBlock>>>(d_A, d_B, d_C, M, N, K);
    cudaDeviceSynchronize();
}

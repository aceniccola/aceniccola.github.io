---
title: "Audio Detection"
permalink: /audio-detect/
layout: single
author_profile: false
---

<style>
  body {
    background: #000;
    color: #0f0;
    font-family: 'Courier New', Consolas, Monaco, monospace;
  }
  .terminal-container {
    background: #000;
    color: #0f0;
    padding: 20px;
    min-height: 400px;
    font-family: 'Courier New', Consolas, Monaco, monospace;
    font-size: 14px;
    line-height: 1.6;
  }
  .terminal-title {
    border-bottom: 1px solid #0f0;
    padding-bottom: 10px;
    margin-bottom: 20px;
    font-size: 18px;
  }
  input[type="file"] {
    background: #000;
    color: #0f0;
    border: 1px solid #0f0;
    padding: 8px;
    font-family: 'Courier New', Consolas, Monaco, monospace;
    margin: 10px 0;
    cursor: pointer;
  }
  button {
    background: #000;
    color: #0f0;
    border: 1px solid #0f0;
    padding: 10px 20px;
    font-family: 'Courier New', Consolas, Monaco, monospace;
    cursor: pointer;
    margin: 10px 0;
    font-size: 14px;
  }
  button:hover {
    background: #0f0;
    color: #000;
  }
  #result {
    margin-top: 20px;
    white-space: pre-wrap;
    word-wrap: break-word;
    border-top: 1px solid #0f0;
    padding-top: 10px;
  }
  .loading {
    animation: blink 1s infinite;
  }
  @keyframes blink {
    0%, 50% { opacity: 1; }
    51%, 100% { opacity: 0; }
  }
</style>

<div class="terminal-container">
  <div class="terminal-title">
    === AUDIO DETECTION TERMINAL ===
  </div>
  
  <div>
    > SELECT AUDIO FILE:
    <br>
    <input type="file" id="audioFile" accept="audio/*">
  </div>
  
  <div>
    <button onclick="detect()">[ DETECT ]</button>
  </div>
  
  <div id="result"></div>
</div>

<script>
  async function detect() {
    const fileInput = document.getElementById('audioFile');
    const resultDiv = document.getElementById('result');
    
    if (!fileInput.files || !fileInput.files[0]) {
      resultDiv.innerHTML = '> ERROR: No file selected';
      return;
    }
    
    const file = fileInput.files[0];
    resultDiv.innerHTML = '> PROCESSING<span class="loading">...</span>';
    
    const formData = new FormData();
    formData.append('file', file);
    
    try {
      const res = await fetch('https://mutable-camryn-arborescently.ngrok-free.dev/detect', {
        method: 'POST',
        body: formData
      });
      
      if (!res.ok) {
        throw new Error(`HTTP ${res.status}: ${res.statusText}`);
      }
      
      const data = await res.json();
      resultDiv.innerHTML = '> RESULT:\n\n' + JSON.stringify(data, null, 2);
    } catch (error) {
      resultDiv.innerHTML = '> ERROR:\n' + error.message + '\n\n> Check API endpoint and CORS settings';
    }
  }
</script>

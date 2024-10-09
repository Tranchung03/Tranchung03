<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GoodNotes-like Drawing App</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      font-family: Arial, sans-serif;
    }

    .toolbar {
      margin: 10px;
    }

    canvas {
      border: 1px solid black;
      background-color: #fff;
      cursor: none;
    }

    #drawingCanvas {
      width: 800px;
      height: 600px;
    }

    .custom-cursor {
      position: absolute;
      width: 15px;
      height: 15px;
      border-radius: 50%;
      background-color: rgba(255, 0, 0, 0.8);
      pointer-events: none;
      display: block;
    }

    .canvas-container {
      position: relative;
      margin-top: 10px;
    }

    .canvas-list {
      margin-top: 10px;
    }

    .canvas-list button {
      margin: 2px;
    }
  </style>
</head>
<body>
  <h1>Draw Like GoodNotes</h1>
  <div class="toolbar">
    <button id="clear">Clear Canvas</button>
    <button id="eraser">Eraser</button>
    <input type="color" id="colorPicker" value="#000000">
    <input type="range" id="brushSize" min="1" max="10" value="5">
    <button id="save">Save as PNG</button>
    <button id="addCanvas">Add New Canvas</button>
  </div>
  <div class="canvas-container" id="canvasContainer"></div>
  <div class="custom-cursor" id="customCursor"></div>
  <div class="canvas-list" id="canvasList"></div>

  <script>
    const canvasContainer = document.getElementById('canvasContainer');
    const customCursor = document.getElementById('customCursor');
    const colorPicker = document.getElementById('colorPicker');
    const brushSizeInput = document.getElementById('brushSize');
    const clearButton = document.getElementById('clear');
    const saveButton = document.getElementById('save');
    const eraserButton = document.getElementById('eraser');
    const addCanvasButton = document.getElementById('addCanvas');
    let currentCanvas = null;
    let drawing = false;
    let brushColor = '#000000';
    let brushSize = 5;
    let isEraser = false;

    const canvases = [];

    function createCanvas() {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');

      canvas.width = 800;
      canvas.height = 600;

      canvas.addEventListener('mousedown', () => {
        drawing = true;
      });

      canvas.addEventListener('mouseup', () => {
        drawing = false;
        ctx.beginPath();
      });

      canvas.addEventListener('mousemove', (e) => {
        const rect = canvas.getBoundingClientRect();
        const mouseX = e.clientX - rect.left;
        const mouseY = e.clientY - rect.top;

        if (drawing) {
          ctx.lineWidth = brushSize;
          ctx.lineCap = 'round';
          ctx.strokeStyle = isEraser ? '#ffffff' : brushColor;

          ctx.lineTo(mouseX, mouseY);
          ctx.stroke();
          ctx.beginPath();
          ctx.moveTo(mouseX, mouseY);
        }

        customCursor.style.left = `${e.clientX - 7.5}px`; // 7.5 để căn giữa dấu chấm đỏ
        customCursor.style.top = `${e.clientY - 7.5}px`; // 7.5 để căn giữa dấu chấm đỏ
      });

      canvasContainer.appendChild(canvas);
      canvases.push(canvas);
      currentCanvas = canvas;

      drawGrid(ctx);
      updateCanvasList();
    }

    function drawGrid(ctx) {
      ctx.strokeStyle = 'rgba(0, 0, 0, 0.1)';
      ctx.lineWidth = 0.5;

      for (let x = 0; x <= ctx.canvas.width; x += 50) {
        ctx.moveTo(x, 0);
        ctx.lineTo(x, ctx.canvas.height);
      }

      for (let y = 0; y <= ctx.canvas.height; y += 50) {
        ctx.moveTo(0, y);
        ctx.lineTo(ctx.canvas.width, y);
      }

      ctx.stroke();
    }

    function updateCanvasList() {
      const canvasList = document.getElementById('canvasList');
      canvasList.innerHTML = '';
      canvases.forEach((canvas, index) => {
        const button = document.createElement('button');
        button.innerText = `Canvas ${index + 1}`;
        button.addEventListener('click', () => {
          currentCanvas = canvas;
          canvases.forEach(c => c.style.display = 'none');
          currentCanvas.style.display = 'block';
        });
        canvasList.appendChild(button);
      });
    }

    colorPicker.addEventListener('input', (e) => {
      brushColor = e.target.value;
      isEraser = false;
    });

    brushSizeInput.addEventListener('input', (e) => {
      brushSize = e.target.value;
    });

    clearButton.addEventListener('click', () => {
      const ctx = currentCanvas.getContext('2d');
      ctx.clearRect(0, 0, currentCanvas.width, currentCanvas.height);
      drawGrid(ctx);
    });

    eraserButton.addEventListener('click', () => {
      isEraser = true;
    });

    saveButton.addEventListener('click', () => {
      const link = document.createElement('a');
      link.download = 'drawing.png';
      link.href = currentCanvas.toDataURL();
      link.click();
    });

    addCanvasButton.addEventListener('click', createCanvas);

    createCanvas(); // Tạo canvas đầu tiên
  </script>
</body>
</html>

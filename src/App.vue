<script setup>
import { ref, onMounted, watch } from "vue";

const canvas = ref(null);
const tabTitle = ref("");
const pixelSize = 20;
const canvasSize = 32 * pixelSize;
const selectedColor = ref("#000000");
const colors = [
  "#000000",
  "#ffffff",
  "#ff0000",
  "#00ff00",
  "#0000ff",
  "#ffff00",
  "#ff00ff",
  "#00ffff",
  "#ff8000",
  "#8000ff",
  "#ff0080",
  "#80ff00",
  "#808080",
  "#800000",
  "#008000",
  "#000080",
];

let ctx = null;
let isDrawing = false;
let canvasData = ref(
  Array(32)
    .fill()
    .map(() => Array(32).fill("#ffffff00")),
);

onMounted(() => {
  ctx = canvas.value.getContext("2d");
  loadFromStorage();
  drawCanvas();

  // Set up canvas event listeners
  canvas.value.addEventListener("mousedown", startDrawing);
  canvas.value.addEventListener("mousemove", draw);
  canvas.value.addEventListener("mouseup", stopDrawing);
  canvas.value.addEventListener("mouseleave", stopDrawing);
});

// Watch for tab title changes and update live
watch(tabTitle, (newTitle) => {
  document.title = newTitle || "Pixel Canvas";
  updateFavicon();
  saveToStorage();
});

// Watch for canvas changes
watch(
  canvasData,
  () => {
    updateFavicon();
    saveToStorage();
  },
  { deep: true },
);

function startDrawing(e) {
  isDrawing = true;
  draw(e);
}

function draw(e) {
  if (!isDrawing) return;

  const rect = canvas.value.getBoundingClientRect();
  const x = Math.floor((e.clientX - rect.left) / pixelSize);
  const y = Math.floor((e.clientY - rect.top) / pixelSize);

  if (x >= 0 && x < 32 && y >= 0 && y < 32) {
    canvasData.value[y][x] = selectedColor.value;
    drawPixel(x, y, selectedColor.value);
  }
}

function stopDrawing() {
  isDrawing = false;
}

function drawCanvas() {
  ctx.fillStyle = "#ffffff00";
  ctx.fillRect(0, 0, canvasSize, canvasSize);

  for (let y = 0; y < 32; y++) {
    for (let x = 0; x < 32; x++) {
      drawPixel(x, y, canvasData.value[y][x]);
    }
  }

  // Draw grid
  ctx.strokeStyle = "#dddddd";
  ctx.lineWidth = 1;
  for (let i = 0; i <= 32; i++) {
    ctx.beginPath();
    ctx.moveTo(i * pixelSize, 0);
    ctx.lineTo(i * pixelSize, canvasSize);
    ctx.stroke();

    ctx.beginPath();
    ctx.moveTo(0, i * pixelSize);
    ctx.lineTo(canvasSize, i * pixelSize);
    ctx.stroke();
  }
}

function drawPixel(x, y, color) {
  ctx.fillStyle = color;
  ctx.fillRect(x * pixelSize, y * pixelSize, pixelSize, pixelSize);
}

function clearCanvas() {
  canvasData.value = Array(32)
    .fill()
    .map(() => Array(32).fill("#ffffff00"));
  drawCanvas();
}

function updateFavicon() {
  const faviconCanvas = document.createElement("canvas");
  faviconCanvas.width = 32;
  faviconCanvas.height = 32;
  const faviconCtx = faviconCanvas.getContext("2d");

  for (let y = 0; y < 32; y++) {
    for (let x = 0; x < 32; x++) {
      faviconCtx.fillStyle = canvasData.value[y][x];
      faviconCtx.fillRect(x, y, 1, 1);
    }
  }

  const dataURL = faviconCanvas.toDataURL();

  let favicon = document.querySelector('link[rel="icon"]');
  if (!favicon) {
    favicon = document.createElement("link");
    favicon.rel = "icon";
    document.head.appendChild(favicon);
  }
  favicon.href = dataURL;
}

function saveToStorage() {
  localStorage.setItem("pixelCanvas", JSON.stringify(canvasData.value));
  localStorage.setItem("tabTitle", tabTitle.value);
}

function loadFromStorage() {
  const savedCanvas = localStorage.getItem("pixelCanvas");
  const savedTitle = localStorage.getItem("tabTitle");

  if (savedCanvas) {
    canvasData.value = JSON.parse(savedCanvas);
  }

  if (savedTitle) {
    tabTitle.value = savedTitle;
    document.title = savedTitle || "Pixel Canvas";
  }
}
</script>

<template>
  <div class="app">
    <h1>32x32 Pixel Canvas</h1>

    <div class="controls">
      <input
        v-model="tabTitle"
        type="text"
        placeholder="Enter tab title..."
        class="title-input"
      />
      <button @click="clearCanvas" class="clear-btn">Clear</button>
    </div>

    <div class="color-palette">
      <div
        v-for="color in colors"
        :key="color"
        :class="['color-swatch', { active: selectedColor === color }]"
        :style="{ backgroundColor: color }"
        @click="selectedColor = color"
      ></div>
    </div>

    <canvas
      ref="canvas"
      :width="canvasSize"
      :height="canvasSize"
      class="pixel-canvas"
    ></canvas>

    <footer class="footer">
      <p>
        Made with ❤️ •
        <a
          href="https://github.com/darkobyte/TabTweak"
          target="_blank"
          rel="noopener noreferrer"
          >Visit GitHub Page</a
        >
      </p>
    </footer>
  </div>
</template>

<style scoped>
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  font-family: Arial, sans-serif;
}

h1 {
  margin-bottom: 20px;
  color: #333;
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  align-items: center;
}

.title-input {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  min-width: 200px;
}

.title-input:focus {
  outline: none;
  border-color: #4caf50;
}

.clear-btn {
  padding: 8px 16px;
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.clear-btn:hover {
  background-color: #da190b;
}

.pixel-canvas {
  border: 2px solid #333;
  cursor: crosshair;
  image-rendering: pixelated;
}

.pixel-canvas:hover {
  border-color: #4caf50;
}

.color-palette {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 4px;
  margin-bottom: 20px;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 8px;
  background-color: #f9f9f9;
}

.color-swatch {
  width: 30px;
  height: 30px;
  border: 2px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.color-swatch:hover {
  transform: scale(1.1);
  border-color: #666;
}

.color-swatch.active {
  border-color: #4caf50;
  border-width: 3px;
  transform: scale(1.1);
}

.footer {
  margin-top: 40px;
  padding: 20px;
  text-align: center;
  border-top: 1px solid #eee;
  color: #666;
  font-size: 14px;
}

.footer a {
  color: #4caf50;
  text-decoration: none;
  transition: color 0.2s ease;
}

.footer a:hover {
  color: #333;
  text-decoration: underline;
}
</style>

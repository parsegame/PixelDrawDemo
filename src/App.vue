<template>
  <div class="box">
    <h1>Pixel Art Demo 当前画布大小 {{ canvasSize }} x {{ canvasSize }}</h1>
    <div class="canvas_container" :style="style">
      <div id="pixiContainer"></div>
    </div>
    <div class="controls">
      <input v-model="input" @keydown.enter="refresh(input)" type="text" style="margin: 10px;" />
      <input v-model="selectedColor" type="color" value="ffffff" style="margin: 10px;" />
      <button @click="clearCanvas">Clear Canvas</button>
    </div>
  </div>
</template>

<script setup lang="ts">

import * as PIXI from 'pixi.js';
import { Color } from 'pixi.js';
import { onMounted, ref } from 'vue';

const pixiApp = ref<PIXI.Application | null>(null);
const pixelSize = 1;
const canvasSize = ref(128 * pixelSize);
const input = ref(128 * pixelSize);
let gridSize = canvasSize.value;
const pixels = Array.from({ length: canvasSize.value }, () => Array(canvasSize.value).fill(''));

let scale = ref(1);

let style = {
  'width': 512 + "px",
  'height': 512 + "px"
}

let selectedColor = '#000000';
let isDrawing = false;
let lastPosition = { x: 0, y: 0 };
let app = ref<PIXI.Application<HTMLCanvasElement>>();

onMounted(() => {
  createCanvans();
});

function createCanvans() {
  scale.value = 512 / canvasSize.value;

  console.log("刷新");
  gridSize = canvasSize.value;

  if (app.value) {
    const oldContainer = document.getElementById('pixiContainer') as HTMLElement;
    oldContainer.removeChild(app.value?.view);
  }

  // 创建一个新的 PIXI.Application 实例
  const newApp = new PIXI.Application<HTMLCanvasElement>({
    width: gridSize,
    height: gridSize,
    background: new Color([0, 0, 0, 0.0]).toArray(),
    resolution: window.devicePixelRatio || 1,
  });

  // 销毁旧的实例
  app.value?.destroy();

  // 使用新的实例
  app.value = newApp;

  // 创建一个 TilingSprite，使用平铺的纹理
  const texture = PIXI.Texture.from('/img/background.png');
  const tilingSprite = new PIXI.TilingSprite(texture, app.value.screen.width, app.value.screen.height);

  // 设置 TilingSprite 的深度，使其置于底层
  tilingSprite.zIndex = 0;
  // 将 TilingSprite 添加到舞台
  app.value.stage.addChild(tilingSprite);

  const container = document.getElementById('pixiContainer') as HTMLElement;
  container.appendChild(app.value.view);

  container.style.transform = "scale(" + scale.value + ")";
  const graphics = new PIXI.Graphics();

  app.value.stage.addChild(graphics);

  const borderGraphics = new PIXI.Graphics(); // 新增的 Graphics 对象
  // app.stage.addChild(graphics);
  app.value.stage.addChild(borderGraphics); // 将边框 Graphics 对象添加到舞台


  app.value.view.addEventListener('mousedown', (event) => {
    isDrawing = true;
    updateLastPosition(event);

  });

  app.value.view.addEventListener('mouseup', () => {
    isDrawing = false;
  });

  app.value.view.addEventListener('mousemove', (event) => {
    if (isDrawing) {
      drawLine(lastPosition.x, lastPosition.y, event.offsetX, event.offsetY);
      updateLastPosition(event);

    }
    updateBorder(event.offsetX, event.offsetY); // 更新边框位置
  });

  pixiApp.value = app.value;
}

// function clamp(value: number, min: number, max: number): number {
//   return Math.min(Math.max(value, min), max);
// }

function refresh(input: number) {
  canvasSize.value = input;
  createCanvans();
}


// function drawPixel(x: number, y: number) {
//   console.log("one pixel", x, y);
//   const graphics = pixiApp.value?.stage.children[0] as PIXI.Graphics;

//   const roundedX = Math.ceil(x);
//   const roundedY = Math.ceil(y);

//   graphics.beginFill(parseInt(selectedColor.replace(/^#/, ''), 16));
//   graphics.drawRect(roundedX - 0.5, roundedY - 0.5, pixelSize, pixelSize);
//   graphics.endFill();
// }

function drawLine(startX: number, startY: number, endX: number, endY: number) {
  const graphics = pixiApp.value?.stage.children[1] as PIXI.Graphics;

  const roundedStartX = Math.ceil(startX);
  const roundedStartY = Math.ceil(startY);
  const roundedEndX = Math.ceil(endX);
  const roundedEndY = Math.ceil(endY);


  console.log("坐标: ", roundedStartX, roundedStartY);

  graphics.lineStyle({ width: 1, color: parseInt(selectedColor.replace(/^#/, ''), 16), native: true });
  graphics.moveTo(roundedStartX, roundedStartY - 1);
  graphics.lineTo(roundedEndX, roundedEndY - 1);
  graphics.closePath();
}

function updateLastPosition(event: MouseEvent) {
  lastPosition = { x: event.offsetX, y: event.offsetY };
}

function updateBorder(mouseX: number, mouseY: number) {
  const borderGraphics = pixiApp.value?.stage.children[2] as PIXI.Graphics;

  borderGraphics.clear();
  borderGraphics.lineStyle({ width: 1, color: parseInt(selectedColor.replace(/^#/, ''), 16), alpha: 0.5 });
  borderGraphics.drawRect(mouseX - 0.5, mouseY - 0.5, pixelSize / 2, 0);
}

function clearCanvas() {
  const graphics = pixiApp.value?.stage.children[1] as PIXI.Graphics;

  graphics.clear();
  pixels.forEach((row, i) => {
    row.forEach((_color, j) => {
      pixels[i][j] = '';
    });
  });
}
</script>


<style scoped>
.canvas_container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.box {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

#pixiContainer {
  display: flex;
  border: 1px solid #ccc;
  /* transform: scale(8); */
  image-rendering: pixelated;
}

.controls {
  margin-top: 20px;
}
</style>

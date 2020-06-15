---
layout: post
title: "[JavaScript30] Day 08 - Canvas"
description: "[JavaScript30] Day 08 - Canvas"
crawlertitle: "[JavaScript30] Day 08 - Canvas"
date: 2020-06-14 22:33:26 +0800
categories: JavaScript
tags: JavaScript
comments: true
---

> This is my note of [JavaScript30](https://javascript30.com/) (*30 day Vanilla JS coding challenge*) -- a fantastic 30 day course teach you to build cool stuffs by just plain JavaScript.  
> - [Here](https://github.com/wesbos/JavaScript30) is the original repository includes starter files and completed solutions from the author [Wes Bos](https://github.com/wesbos).

---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Target
Having fun with html `<canvas>` element.

## Key points
### Canvas API
The Canvas API provides a means for drawing graphics via JavaScript and HTML `<canvas>` element. It can be used for animation, game graphics, data visualization, photo manipulation, and real-time video processing.
> [MDN web docs](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)

**HTML**
```html
<canvas id="draw" width="800" height="800"></canvas>
```

**JavaScript**
The `canvas` element has bunch of properties that can be configured.
- getContext
  ```js
  const ctx = canvas.getContext('2d');

  ctx.strokeStyle = '#BADA55';
  ctx.lineJoin = 'round';
  ctx.lineCap = 'round';
  ctx.lineWidth = 10;
  ```

### HSL
Cylindrical geometries:
![HSL cylinder from Wikipedia](https://upload.wikimedia.org/wikipedia/commons/thumb/6/6b/HSL_color_solid_cylinder_saturation_gray.png/640px-HSL_color_solid_cylinder_saturation_gray.png)
> HSL cylinder from [Wikipedia](https://en.wikipedia.org/wiki/HSL_and_HSV)

- hue: The angular dimension starting at the red at 0°, passing through the treen primary at 120° and the blue primary at 240°, and thenwrapping back to red at 360°
- saturation: 0 ~ 100%
- lightness: 0 ~ 100%

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Demostration
> **Note:** Resizing browser will clear content.
<div id="canvas-wrapper" style="height: 600px;">
  <div class="demo-btn-wrapper" style="display: flex; justify-content: space-between;">
    <div>
      <label for="line-size">Line Size</label>
      <input id="line-size" type="range" name="line-size" min="5" max="100" value="50">
    </div>
    <div>
      <button id="pointer" type="button">Earser</button>
      <button id="clear-btn" type="button">Clear</button>
    </div>
  </div>
  <canvas id="draw" style="border: 1px solid gray" height="500"></canvas>
</div>
<script>
  const hasTouchEvent = 'ontouchstart' in window ? true : false;
  const downEvent = hasTouchEvent ? 'touchstart' : 'mousedown';
  const moveEvent = hasTouchEvent ? 'touchmove' : 'mousemove';
  const upEvent = hasTouchEvent ? 'touchend' : 'mouseup';
  const lineSize = document.getElementById('line-size');
  const pointer = document.getElementById('pointer');
  const clearBtn = document.getElementById('clear-btn');
  const demo = document.querySelector('#demo');
  const canvas = document.querySelector('#draw');
  const ctx = canvas.getContext('2d');
  const canvasWrapper = document.getElementById('canvas-wrapper');
  canvas.width = canvasWrapper.offsetWidth;
  canvas.height = canvasWrapper.offsetHeight;

  let isEarser = false;
  let isDrawing = false;
  let lastX = 0;
  let lastY = 0;
  let hue = 0;
  let size = lineSize.value;

  const position = (e) => {
    var x, y;

    if (hasTouchEvent) {
      x = e.touches[0].pageX - e.target.offsetLeft;
      y = e.touches[0].pageY - e.target.offsetTop;
    } else {
      x = e.offsetX;
      y = e.offsetY;
    }
    return { x: x, y: y };
  };

  function draw(e) {
    if (!isDrawing) return;
    let movePosition = position(e);
    ctx.strokeStyle = `hsl(${hue}, 100%, 50%)`;
    ctx.beginPath();
    ctx.lineWidth = size;
    ctx.moveTo(lastX, lastY);

    ctx.lineTo(movePosition.x, movePosition.y);
    ctx.stroke();
    [lastX, lastY] = [movePosition.x, movePosition.y];

    hue++;
    if (hue >= 360) {
      hue = 0;
    };

    e.preventDefault();
  };

  function pointerHandler() {
    if (isEarser) {
      ctx.globalCompositeOperation = 'color';
      ctx.lineWidth = 50;
      pointer.innerHTML = 'Painter';
      isEarser = false;
    } else {
      ctx.globalCompositeOperation = 'destination-out';
      pointer.innerHTML = 'Earser';
      isEarser = true;
    };
  };

  function resizeCanvas(){
    console.log('resiiiizing~');
    canvas.width = canvasWrapper.offsetWidth;
    canvas.height = canvasWrapper.offsetHeight - clearBtn.offsetHeight;
    ctx.strokeStyle = '#fa0';
    ctx.lineJoin = 'round';
    ctx.lineCap = 'round';
    ctx.lineWidth = 50;
    ctx.globalCompositeOperation = 'color';
    ctx.miterLimit = 100;
  };

  function updateSize(e) {
    size = e.target.value;
  };

  const clearCanvas = () => {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  };

  canvas.addEventListener(downEvent, (e) => {
    isDrawing = true;
    [lastX, lastY] = [e.offsetX, e.offsetY];
  });

  canvas.addEventListener(moveEvent, draw);
  canvas.addEventListener(upEvent, () => isDrawing = false);
  lineSize.addEventListener('input', updateSize);
  pointer.addEventListener('click', pointerHandler);
  clearBtn.addEventListener('click', clearCanvas);
  window.addEventListener('resize', resizeCanvas);
  resizeCanvas();
</script>
---

## Reference
- [HLS and HSV](https://en.wikipedia.org/wiki/HSL_and_HSV) - Wikipedia
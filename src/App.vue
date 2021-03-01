<template>
  <div id="app">
    <canvas ref="canvas" @contextmenu.prevent></canvas>
  </div>
</template>

<script>
import * as PIXI from 'pixi.js';

let points = [[50, 50], [300, 300], [500, 50]];
let app, container, containerLines, containerDisks;
const symbolData = Symbol();
const symbolDragging = Symbol();
const symbolIndex = Symbol();

export default {
  name: 'App',
  mounted() {
    this.initializeCanvas();
    this.drawLines();
    this.drawDisks();
  },
  beforeDestroy() {
    app.destroy(true, true);
    containerLines = null;
    containerDisks = null;
    container = null;
    app = null;
  },
  methods: {
    calculatePoints(x1, y1, x2, y2) {
      const angle = Math.atan((x2 - x1) / (y2 - y1));
      const offset = 1.4142135623730951;
      const offsetX = offset * Math.cos(angle);
      const offsetY = offset * Math.sin(angle);
      const points = [
        x1 + offsetX, y1 - offsetY,
        x2 + offsetX, y2 - offsetY,
        x2 - offsetX, y2 + offsetY,
        x1 - offsetX, y1 + offsetY
      ];
      return points;
    },
    changeLineStyleColor(event, newColor) {
      let graphic = event.currentTarget;
      if (graphic && graphic.geometry.graphicsData[0]) {
        graphic.geometry.graphicsData[0].lineStyle.color = newColor;
        graphic.geometry.invalidate();
      }
    },
    changeFillStyleColor(event, newColor) {
      let graphic = event.currentTarget;
      if (graphic && graphic.geometry.graphicsData[0]) {
        graphic.geometry.graphicsData[0].fillStyle.color = newColor;
        graphic.geometry.invalidate();
      } 
    },
    drawDisk(x, y, index) {
      const that = this;
      const onDragStart = function(event) {
        if (event.data.originalEvent.button === 0) {
          this[symbolData] = event.data;
          this[symbolDragging] = true;
        }
      };
      const onDragEnd = function() {
        this[symbolData] = null;
        this[symbolDragging] = false;
      };
      const onDragMove = function() {
        if (this[symbolDragging]) {
          const {x, y} = this[symbolData].getLocalPosition(this.parent);
          this.x = x;
          this.y = y;
          points[index][0] = x;
          points[index][1] = y;
          that.redrawLines();
        }
      };
      const onRightClick = function() {
        const index = this[symbolIndex];
        if (index > 0 && index < points.length - 1) {
          points.splice(index, 1);
          that.redrawLinesAndDisk();
        }
      };
      let gr = new PIXI.Graphics();
      gr[symbolIndex] = index;
      gr.buttonMode = true;
      gr.interactive = true;
      gr.x = x;
      gr.y = y;
      gr.beginFill(0x00ff00)
        .drawCircle(0, 0, 10)
        .endFill()
        .on('mouseover', event => {this.changeFillStyleColor(event, 0x00ff00)})
        .on('mouseout', event => {this.changeFillStyleColor(event, 0x66ff00)})
        .on('mousedown', onDragStart)
        .on('mouseup', onDragEnd)
        .on('mouseupoutside', onDragEnd)
        .on('mousemove', onDragMove)
        .on('rightclick', onRightClick);
      return gr;
    },
    drawDisks() {
      for (let i = 0; i < points.length; i++) {
        const disk = this.drawDisk(...points[i], i);
        containerDisks.addChild(disk);
      }
    },
    drawLine(x1, y1, x2, y2, index) {
      const that = this;
      const pointsPolygon = this.calculatePoints(x1, y1, x2, y2);
      const onClick = function(event) {
        if (event.data.originalEvent.button === 0) {
          const {x, y} = event.data.getLocalPosition(this.parent);
          points.splice(this[symbolIndex], 0, [x, y]);
          that.redrawLinesAndDisk();
        }
      };
      let gr = new PIXI.Graphics();
      gr[symbolIndex] = index;
      gr.buttonMode  = true;
      gr.hitArea = new PIXI.Polygon(pointsPolygon);
      gr.interactive = true;
      gr.lineStyle(1, 0xff0000, 1, .5, true)
        .moveTo(x1, y1)
        .lineTo(x2, y2)
        .on('click', onClick)
        .on('mouseover', event => {this.changeLineStyleColor(event, 0xff7f50)})
        .on('mouseout', event => {this.changeLineStyleColor(event, 0xff0000)});
      return gr;
    },
    drawLines() {
      for (let i = 1; i < points.length; i++) {
        const line = this.drawLine(...points[i-1],...points[i], i);
        containerLines.addChild(line);
      }
    },
    initializeCanvas() {
      app = new PIXI.Application({
        view: this.$refs.canvas,
        width: window.innerWidth, 
        height: window.innerHeight,
        antialias: true,
        autoDensity: true,
        backgroundColor: 0xF6F6F6,
        resolution: window.devicePixelRatio || 1,
        powerPreference: 'high-performance'
      });
      
      container = new PIXI.Container();
      containerLines = new PIXI.Container();
      containerDisks = new PIXI.Container();

      container.addChild(containerLines);
      container.addChild(containerDisks);
      app.stage.addChild(container);
    },
    redrawDisk() {
      containerDisks.removeChildren();
      this.drawDisks();
    },
    redrawLines() {
      containerLines.removeChildren();
      this.drawLines();
    },
    redrawLinesAndDisk() {
      this.redrawLines();
      this.redrawDisk();
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}
#app canvas {
  display: block;
}
</style>

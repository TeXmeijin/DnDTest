<template>
  <div class="container">
    <h1>お絵かき</h1>
    <div class="render-area">
      <div
        class="render-area__inner"
        @mousedown="onMouseDown"
        @mousemove="onMouseMove"
        @mouseup="onMouseUp"
      >
        <div v-for="obj in objects" :key="obj.id" :style="getStyle(obj)"></div>
        <div
          v-for="obj in drawingObjects"
          :key="obj.id"
          :style="getStyle(obj)"
        ></div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { v4 as uuidv4 } from 'uuid'

const px = (px: number) => `${px}px`

const min = (a: number, b: number) => {
  if (a > b) return b
  return a
}

const realXY = (evt: MouseEvent) => {
  const target = evt.currentTarget! as Element
  const targetRect = target.getBoundingClientRect()

  return {
    x: evt.clientX - targetRect.left,
    y: evt.clientY - targetRect.top,
  }
}

type RectanglePosition = {
  x1: number
  y1: number
  x2: number
  y2: number
}

type Rectangle = {
  position: RectanglePosition
  color: {
    border: string
    background: string
  }
}

type DrawingObject = Rectangle & { id: string }

export default Vue.extend({
  data() {
    return {
      objects: [] as DrawingObject[],
      drawingObjects: [] as DrawingObject[],
    }
  },
  methods: {
    onMouseDown(evt: MouseEvent) {
      const { x, y } = realXY(evt)
      this.drawingObjects.push({
        id: uuidv4(),
        position: {
          x1: x,
          y1: y,
          x2: x + 16,
          y2: y + 16,
        },
        color: {
          border: '#333',
          background: '#aaa',
        },
      })
    },
    onMouseMove(evt: MouseEvent) {
      const { x, y } = realXY(evt)
      this.drawingObjects.forEach((obj) => {
        obj.position.x2 = x
        obj.position.y2 = y
      })
    },
    onMouseUp() {
      this.objects.push(...this.drawingObjects)
      this.drawingObjects.splice(0, this.drawingObjects.length)
    },
    getStyle(obj: DrawingObject): Partial<CSSStyleDeclaration> {
      return {
        position: 'absolute',
        left: px(min(obj.position.x1, obj.position.x2)),
        top: px(min(obj.position.y1, obj.position.y2)),
        width: px(Math.abs(obj.position.x1 - obj.position.x2)),
        height: px(Math.abs(obj.position.y1 - obj.position.y2)),
        backgroundColor: `${obj.color.background}`,
        borderWidth: px(1),
        borderColor: obj.color.border,
        borderStyle: 'solid',
      }
    },
  },
})
</script>

<style lang="scss" scoped>
.render-area {
  display: flex;
  width: 100%;
  min-height: 100vh;
  background: #f0f0f0;

  &__inner {
    flex: 1;
    min-height: 100vh;
    position: relative;
  }
}
</style>

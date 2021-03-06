<template>
  <div class="container margins" id="playground">
    <div class="row">
      <div class="col-md-8 col-xs-8">
        <Canvas ref="canvas"/>
      </div>
      <div class="col-md-4 col-xs-8">
        <Tool
          v-for="(tool, id) of tools"
          @select-tool="currentTool = $event"
          :key="id"
          :name="tool.name"
          :imageUrl="tool.imageUrl"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Canvas from './playground/Canvas'
import Tool from './playground/Tool'
import Circle from '../algorithms/circle'
import Ellipse from '../algorithms/ellipse'
import Line from '../algorithms/bresenham'
import CohenSutherland from '../algorithms/csclip'
import MemoryBuffer from '../algorithms/memory-buffer'
import FloodFill from '../algorithms/floodfill'
import { arrayReplace, randomHexColor } from '../algorithms/utils'
import { translate2d, rotate2d, scale2d } from '../algorithms/transform'
import { orthographicXY } from '../algorithms/projections'
import ScanLineFill from '../algorithms/scan-line-fill'
import MidpointClip from '../algorithms/mpclip'

export default {
  name: 'Playground',
  components: {
    Canvas,
    Tool
  },
  data: () => ({
    tools: [
      {
        name: 'Line',
        imageUrl: 'http://lorempixel.com/output/abstract-q-c-16-16-5.jpg'
      },
      {
        name: 'Circle',
        imageUrl: 'http://lorempixel.com/output/abstract-q-c-16-16-5.jpg'
      }
    ],
    currentTool: '',
    zBuffer: []
  }),

  methods: {
    setCurrentTool (toolName) {
      this.currentTool = toolName
    }

    // onClickPixel (shape. x. y)
  },

  mounted () {
    this.canvas = this.$refs.canvas
    this.buffer = new MemoryBuffer(this.canvas.size)

    const ellipse = new Ellipse(this.buffer, '#0066ff', {
      centerPoint: [25, 35],
      a: 15,
      b: 8
    })
    this.zBuffer.push(ellipse)

    const circle = new Circle(this.buffer, '#800000', {
      centerPoint: [7, 21],
      radius: 5
    })
    this.zBuffer.push(circle)

    const circle2 = new Circle(this.buffer, '#800000', {
      centerPoint: [43, 21],
      radius: 5
    })
    this.zBuffer.push(circle2)

    const line = new Line(this.buffer, '#cdcf7f', {
      fromPoint: [45, 15],
      toPoint: [10, 41]
    })
    this.zBuffer.push(line)

    const flood = new FloodFill(this.buffer, '#cd4001', {
      initPoint: [25, 35]
    })
    this.zBuffer.push(flood)

    // // generate rect for scanline
    // const rectBuffer = new MemoryBuffer(this.canvas.size)
    // const rectPoints = [
    //   [25, 2],
    //   [25, 12],
    //   [35, 12],
    //   [35, 2]
    // ]
    // const rectLines = rectPoints.map(([x1, y1], i) => {
    // const [x2, y2] = rectPoints[(i + 1) % 4]

    //   const line = new Line(rectBuffer, '#cdcf7f', {
    //     fromPoint: [x1, y1],
    //     toPoint: [x2, y2]
    //   })

    //   line.draw()

    //   return line
    // })
    // this.canvas.readMemoryBuffer(rectBuffer)

    // const scanlinefill = new ScanLineFill(this.buffer, rectBuffer, '#cd4001')
    // this.zBuffer.push(scanlinefill)

    // // generate polygon for scanline
    // const polygonBuffer = new MemoryBuffer(this.canvas.size)
    // const polygonPoints = [
    //   [40, 46],
    //   [48, 49],
    //   [52, 44],
    //   [53, 52],
    //   [41, 52]
    // ]
    // const polygonLines = polygonPoints.map(([x1, y1], i) => {
    // const [x2, y2] = polygonPoints[(i + 1) % 5]

    //   const line = new Line(polygonBuffer, '#cdcf7f', {
    //     fromPoint: [x1, y1],
    //     toPoint: [x2, y2]
    //   })

    //   line.draw()

    //   return line
    // })
    // this.canvas.readMemoryBuffer(polygonBuffer)

    // const scanlinefill2 = new ScanLineFill(this.buffer, polygonBuffer, '#cd4001')
    // this.zBuffer.push(scanlinefill2)

    const clipper = new CohenSutherland({
      xmax: 41,
      ymax: 42,
      xmin: 13,
      ymin: 20
    })

    const newLinePts = clipper.clip(line.getLine())
    const newLine = new Line(this.buffer, '#cdcf7f', {
      fromPoint: newLinePts[0],
      toPoint: newLinePts[1]
    })
    arrayReplace(this.zBuffer, line, newLine)

    // let newLinePts2 = newLine.getLine()
    // const [x, y] = newLinePts2[0]
    // newLinePts2 = newLinePts2.map(pt => rotate2d(pt, [x, y], 15))
    // const newLine2 = new Line(this.buffer, '#e4e4e4', {
    //   fromPoint: newLinePts2[0],
    //   toPoint: newLinePts2[1]
    // })
    // this.zBuffer.push(newLine2)

    // // translation
    // const translation = [10, 10]
    // const translatedLine = new Line(this.buffer, randomHexColor(), {
    //   fromPoint: translate2d([0, 20], translation),
    //   toPoint: translate2d([20, 0], translation)
    // })
    // this.zBuffer.push(translatedLine)

    // // scale line
    // const scale = [1.5, 1.5]
    // const scaledLine = new Line(this.buffer, randomHexColor(), {
    //   fromPoint: scale2d([0, 20], scale),
    //   toPoint: scale2d([20, 0], scale)
    // })
    // this.zBuffer.push(scaledLine)

    const line3d = [[0, 0, 0, 1], [2, 2, 2, 1]]
    const projLinePts = line3d.map(point => orthographicXY(point).slice(0, 2))
    console.log(projLinePts)
    const projLine = new Line(this.buffer, '#e4e4e4', {
      fromPoint: projLinePts[0],
      toPoint: projLinePts[1]
    })
    this.zBuffer.push(projLine)
  },

  watch: {
    zBuffer: function () {
      this.zBuffer.forEach(shape => shape.draw())
      this.canvas.readMemoryBuffer(this.buffer)
    }
  }
}
</script>

<style scoped>
.margins {
  margin-top: 50px;
}

.bordered {
  border: 1px solid red;
}
</style>

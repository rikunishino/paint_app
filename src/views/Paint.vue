<template>
  <div>
    <canvas ref="paintArea" class="paintArea" @mousemove="draw" @mousedown="drawStart" @mouseup="drawEnd" width="400" height="400"></canvas>
    <p>
    {{ msg }}
    </p>
    <button class="clearCanvasButton" @click="clearCanvas">クリア</button>
  </div>
</template>

<script>
// マウスカーソルの座標
var x = 0;
var y = 0;

// ドラッグしているか
var isDrag = false;

export default {
  data() {
    return {
      msg: 'Paint Area'
    }
  },
  methods: {
    /**
     * 描画メソッド
     */
    draw: function() {
      // dragしていない時は描画しない
      if(!isDrag) {
        return
      }
      // console.log('draw')

      // canvasの取得
      const _canvas = this.$refs.paintArea
      const _context = _canvas.getContext('2d')

      // canvasの座標取得
      const canvasPosX = _canvas.getBoundingClientRect().left
      const canvasPosY = _canvas.getBoundingClientRect().top

      // canvas内でのマウスカーソルの座標取得
      x = event.pageX - canvasPosX
      y = event.pageY - canvasPosY
      // console.log(x)
      // console.log(y)

      // TODO: 描画設定
      _context.strokeStyle = 'black'
      _context.lineWidth = '10'
      _context.lineCap = 'round'

      // 描画処理
      _context.beginPath()
      _context.moveTo(x, y)
      _context.lineTo(x, y)
      _context.stroke()
    },
    /**
     * 描画開始
     */
    drawStart: function() {
      // console.log('start')
      isDrag = true
    },
    /**
     * 描画終了
     */
    drawEnd: function() {
      // console.log('end')
      isDrag = false
    },
    /**
     * canvasのクリア
     */
    clearCanvas: function() {
      // console.log('clear')
      // canvasの取得
      const _canvas = this.$refs.paintArea
      const _context = _canvas.getContext('2d')

      // クリア処理
      _context.clearRect(0, 0, _canvas.width, _canvas.height)
    }
  }
}
</script>

<style>
.paintArea {
  border: solid 3px #000000;
}
</style>
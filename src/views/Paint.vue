<template>
  <div>
    <canvas ref="paintArea" class="paintArea" @mousemove="draw" @mousedown="drawStart" @mouseup="drawEnd" width="400" height="400"></canvas>
    <p>
    {{ msg }}
    </p>
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
      console.log('draw')

      // 要素取得
      const element = this.$refs.paintArea
      const context = element.getContext('2d')

      // canvasの座標取得
      const canvasPosX = element.getBoundingClientRect().left
      const canvasPosY = element.getBoundingClientRect().top

      // canvas内でのマウスカーソルの座標取得
      x = event.pageX - canvasPosX
      y = event.pageY - canvasPosY
      // console.log(x)
      // console.log(y)

      // TODO: 描画設定
      context.strokeStyle = 'black'
      context.lineWidth = '10'
      context.lineCap = 'round'

      // 描画処理
      context.beginPath()
      context.moveTo(x, y)
      context.lineTo(x, y)
      context.stroke()
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
    }
  }
}
</script>

<style>
.paintArea {
  border: solid 3px #000000;
}
</style>
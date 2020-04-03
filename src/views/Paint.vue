<template>
  <div>
    <canvas ref="paintArea" class="paintArea" @mousemove="draw" @mousedown="drawStart" @mouseup="drawEnd" width="400" height="400"></canvas>
    <p>
    {{ msg }}
    </p>
    <table class=colorTable>
      <caption>Color</caption>
      <tr>
        <td class="black" @click="setColor('black')"></td>
        <td class="white" @click="setColor('white')"></td>
      </tr>
      <tr>
        <td class="red" @click="setColor('red')"></td>
        <td class="blue" @click="setColor('blue')"></td>
      </tr>
      <tr>
        <td class="yellow" @click="setColor('yellow')"></td>
        <td class="green" @click="setColor('green')"></td>
      </tr>
    </table>
    <div class="size">
      <span class="sizeText">
        Size
      </span>
      <input v-model="size" type="range" step="0.1" />
          <span class="sizeValueText">
        {{ size }}
      </span>
    </div>
    <button class="clearCanvasButton" @click="clearCanvas">クリア</button>
    <button class="clearCanvasButton" @click="eraser">消しゴム</button>
  </div>
</template>

<script>
// マウスカーソルの座標
var x = 0;
var y = 0;

// ドラッグしているか
var isDrag = false;

/**
 * ペンの設定値
 *
 * デフォルト値
 * selectedColor(色): 黒
 * selectedSize(太さ): 10
 */
var selectedColor = 'black' // 選択中の色
var selectedSize = '10' // 指定したサイズ


export default {
  data() {
    return {
      msg: 'Paint Area',
      // カラーバリエーション
      colorVariations: [
        ['black', 'white'],
        ['red', 'blue'],
        ['yellow', 'green']
      ],
      // 太さ
      size: selectedSize
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

      // デフォルト描画設定
      _context.strokeStyle = selectedColor
      _context.lineWidth = this.size
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
     * ペンの色設定
     */
    setColor: function(color) {
      // 色の変更
      selectedColor = color
    },
    /**
     * 消しゴム
     */
    eraser: function() {
      selectedColor = 'white'
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
/*
    描画範囲
*/
.paintArea {
  border: solid 3px #000000;
}


/*
    カラーテーブル
*/
.colorTable td {
  width: 40px;
  height: 20px;
}

.colorTable .black {
  background-color: black;
  border: solid 0.5px #000000;
}

.colorTable .white {
  background-color: white;
  border: solid 0.5px #000000;
}

.colorTable .red {
  background-color: red;
  border: solid 0.5px #000000;
}

.colorTable .blue {
  background-color: blue;
  border: solid 0.5px #000000;
}

.colorTable .yellow {
  background-color: yellow;
  border: solid 0.5px #000000;
}

.colorTable .green {
  background-color: green;
  border: solid 0.5px #000000;
}

/*
    文字サイズ
*/
.size {
  text-align: left;
}

.size .sizeText, .size .sizeValueText {
  margin-left: 10px;
  margin-right: 10px;
}
</style>
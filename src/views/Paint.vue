<template>
  <div class="paintApp" @mouseup="drawEnd">
    <div class="leftMenu">
      <canvas ref="penPreview" class="penPreview" width="130" height="110"></canvas>
      <div class="size">
        <span class="sizeText">
          Size
        </span>
        <input @input="drawPreview" v-model="size" type="range" step="0.1" />
            <span class="sizeValueText">
          {{ size }}
        </span>
      </div>
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
      <button class="penButton" @click="changePenType('pen')">ペン</button>
      <button class="eraserButton" @click="changePenType('eraser')">消しゴム</button>
      <button class="clearCanvasButton" @click="clearCanvas">クリア</button>
    </div>
    <canvas ref="paintArea" class="paintArea" @mousemove="draw" @mousedown="drawStart" width="800" height="600"></canvas>
  </div>
</template>

<script>
// canvas
var _canvas = null
var _context = null
// canvasの座標
var canvasPosX = 0
var canvasPosY = 0

// canvas(プレビュー)
var _pCanvas = null
var _pContext = null

// プレピューの表示位置
var pPosX = 0
var pPosY = 0

// マウスカーソルの座標
var posX = 0
var posY = 0

// 前回draw呼び出し時のマウスカーソルの座標
var beforePosX = 0
var beforePosY = 0

// ドラッグしているか
var isDrag = false

/**
 * ペンの設定値
 *
 * デフォルト値
 * selectedColor(色): 黒
 * selectedSize(太さ): 10
 */
var selectedColor = 'black' // 選択中の色
var selectedSize = '10' // 指定したサイズ

/**
 * 消しゴムの設定値
 *
 * デフォルト値
 * selectedSizeEraser(太さ): 10
 */
var selectedSizeEraser = '10' // 指定したサイズ

// ペン or 消しゴム
var penType = 'pen'
penTypeList: [
  'pen', 'eraser'
]

export default {
  data() {
    return {
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
  mounted: function(){
    this.init()
    this.drawPreview()
  },
  methods: {
    /**
     * 初期設定
     */
    init: function() {
      // canvasの取得
      _canvas = this.$refs.paintArea
      _context = _canvas.getContext('2d')
      // canvasの座標取得
      canvasPosX = _canvas.getBoundingClientRect().left
      canvasPosY = _canvas.getBoundingClientRect().top

      // canvas(プレビュー)の取得
      _pCanvas = this.$refs.penPreview
      _pContext = _pCanvas.getContext('2d')

      // プレピューの表示位置を取得
      pPosX = (_pCanvas.getBoundingClientRect().right - _pCanvas.getBoundingClientRect().left) / 2
      pPosY = (_pCanvas.getBoundingClientRect().bottom - _pCanvas.getBoundingClientRect().top) / 2
    },
    /**
     * ペンの設定
     */
    penSetting: function() {
      switch(penType) {
        case 'pen':
          console.log('pen')
          _context.strokeStyle = selectedColor
          selectedSize = this.size
          _context.lineWidth = selectedSize
          _context.lineCap = 'round'
          break
        case 'eraser':
          console.log('eraser')
          _context.strokeStyle = 'white'
          selectedSizeEraser = this.size
          _context.lineWidth = selectedSizeEraser
          _context.lineCap = 'round'
      }
    },
    /**
     * 描画メソッド
     */
    draw: function() {
      // dragしていない時は描画しない
      if(!isDrag) {
        return
      }

      // canvas内でのマウスカーソルの座標取得
      posX = event.pageX - canvasPosX
      posY = event.pageY - canvasPosY

      // 描画処理
      _context.beginPath()
      if((posX === beforePosX) && (posY === beforePosY)) {
        _context.moveTo(posX, posY)
      } else {
        _context.moveTo(beforePosX, beforePosY)
      }
      _context.lineTo(posX, posY)
      _context.stroke()

      //座標更新
      beforePosX = posX
      beforePosY = posY
    },
    /**
     * 描画開始
     */
    drawStart: function() {
      // console.log('start')
      isDrag = true

      // canvas内でのマウスカーソルの座標取得
      posX = event.pageX - canvasPosX
      posY = event.pageY - canvasPosY

      // 座標更新
      beforePosX = posX
      beforePosY = posY

      // ペンの設定
      this.penSetting()
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
     * ペン切り替え
     */
    changePenType: function(type) {
      penType = type

      // サイズ切り替え
      switch(penType) {
        case 'pen':
          this.size = selectedSize
          break
        case 'eraser':
          this.size = selectedSizeEraser
          break
      }

      // プレビューの更新
      this.drawPreview()
    },
    /**
     * canvasのクリア
     */
    clearCanvas: function() {
      // クリア処理
      _context.clearRect(0, 0, _canvas.width, _canvas.height)
    },
    /**
     * プレビューの描画
     */
    drawPreview: function() {
      // クリア処理
      _pContext.clearRect(0, 0, _pCanvas.width, _pCanvas.height)

      // 描画処理
      _pContext.beginPath()
      _pContext.arc( pPosX, pPosY, this.size / 2, 0, 360 , false )
      _pContext.fillStyle = "black"
      _pContext.fill()
    }
  }
}
</script>

<style>
/*
    親要素
 */
.paintApp {
  display: flex;
}

/*
    左メニュー
*/
.leftMenu {
  width: 260px;
  margin-right: 10px;
  border: solid 3px #000000;
}

/*
    プレビュー範囲
*/
.penPreview {
  margin: 10px;
  border: solid 3px #000000;
}

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
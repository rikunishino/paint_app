<template>
  <div ref="purchasePage" class="purchasePage" @mousemove="dragging" @mouseup="reset">

    <div class="product">
      <select v-model="subject">
        <option>国語</option>
        <option>算数</option>
        <option>英語</option>
        <option>理科</option>
        <option>社会</option>
      </select>
      {{ products.subject }}
      <ul class="productList">
        <li v-for="product in products" :key="product.id" @mousedown="dragStart(product, $event)">
          <div ref="product" v-if="product.subject==subject">
            商品ID: {{ product.id }} - 商品名: {{ product.name }} - 値段: {{ product.price }}円
          </div>
        </li>
      </ul>
      <!-- <div ref="aaa">aaa</div> -->
      <button @click="move()">move</button>
    </div>

    <div class="cart" @mouseup="dragEnd">
      <ul class="putProductList">
        <li v-for="putProduct in putProducts" :key="putProduct.id">
          商品ID: {{ putProduct.id }} - 商品名: {{ putProduct.name }} - 値段: {{ putProduct.price }}円 - 個数: {{ putProduct.amount }}個
        </li>
      </ul>
      <div class="total">
        合計: {{ total }}(税込: {{ totalIncludedTax }})円
      </div>
    </div>
  </div>
</template>

<script>
// 選択中の科目
var selectedSubject = '国語';

// 商品
var product = null
// 商品の要素
var productElement = null
// アニメーション用の商品要素クローン
var productClone = null

// 購入ページの要素
var purchasePageElement = null

// ドラッグしているかどうか
var isDrag = false

// 税率
var TAX_RATE = 0.1

export default {
  data() {
    return {
      // 商品一覧
      products: [
        {id: '0', sort_num: 0, subject: '国語', name: '国語の教科書', price: 2400, amount: 0},
        {id: '1', sort_num: 1, subject: '国語', name: '国語のドリル', price: 1500, amount: 0},
        {id: '2', sort_num: 2, subject: '国語', name: '国語のノート', price: 700, amount: 0},
        {id: '3', sort_num: 0, subject: '算数', name: '算数の教科書', price: 2600, amount: 0},
        {id: '4', sort_num: 1, subject: '算数', name: '算数のドリル', price: 1800, amount: 0},
        {id: '5', sort_num: 2, subject: '算数', name: '算数のドリル2', price: 1600, amount: 0},
        {id: '6', sort_num: 3, subject: '算数', name: '算数のノート', price: 600, amount: 0},
        {id: '7', sort_num: 0, subject: '英語', name: '英語の教科書', price: 1200, amount: 0},
        {id: '8', sort_num: 1, subject: '英語', name: '英語のドリル', price: 1100, amount: 0},
        {id: '9', sort_num: 2, subject: '英語', name: '英語のノート', price: 700, amount: 0},
        {id: '10', sort_num: 0, subject: '理科', name: '理科の教科書', price: 3000, amount: 0},
        {id: '11', sort_num: 1, subject: '理科', name: '理科のノート', price: 900, amount: 0},
        {id: '12', sort_num: 0, subject: '社会', name: '社会の教科書', price: 2800, amount: 0},
        {id: '13', sort_num: 1, subject: '社会', name: '社会のノート', price: 800, amount: 0}
      ],
      // 表示中の科目
      subject: selectedSubject,

      // カート内商品
      putProducts: [],

      // 合計金額
      total: 0,
      // 税込
      totalIncludedTax: 0
    }
  },
  mounted: function() {
    // 商品の要素取得
    productElement = this.$refs.product
    purchasePageElement = this.$refs.purchasePage
  },
  methods: {
    /**
     * ドラッグ開始
     */
    dragStart: function(targetProduct) {
      // console.log('ドラッグ開始')
      // フラグ更新
      isDrag = true

      // 選択された商品をセット
      product = targetProduct
      console.log(product.sort_num)
      // アニメーション用のクローン作成
      productClone = productElement[product.sort_num].cloneNode(true)
      productClone.style.position = 'absolute'
      productClone.style.top = event.pageY + 10 + 'px'
      productClone.style.left = event.pageX + 10 + 'px'
      purchasePageElement.appendChild(productClone)
    },
    /**
     * ドラッグ中
     */
    dragging: function(product) {
      if(!isDrag) {
        return
      }
      productClone.style.top = event.pageY + 10 + 'px'
      productClone.style.left = event.pageX + 10 + 'px'
    },
    /**
     * ドラッグ終了（カート外でドラッグを終了した場合は呼ばれない）
     */
    dragEnd: function() {
      if(!isDrag) {
        return
      }
      console.log('ドラッグ終了')
      isDrag = false



      var index = this.checkDuplication(product.id)

      // すでにカートにある場合は個数を増加
      if(index >= 0) {
        this.putProducts[index].amount += 1
      } else {
        // カートに商品を追加
        this.putProducts[this.putProducts.length] =
          { id: product.id, subject: product.subject, name: product.name, price: product.price, amount:product.amount + 1 }
      }

      //合計金額算出
      this.totalPrice()
    },
    /**
     * ドラッグ終了時のリセット処理
     */
    reset: function() {
      product = null
      isDrag = false
      if(productClone != null) {
        productClone.remove()
      }
    },
    /**
     * カート内の商品と重複をチェック
     */
    checkDuplication: function(id) {
      for(var i = 0; i < this.putProducts.length; i++) {
        // console.log(id + ':' + this.putProducts[i].id)
        if(id === this.putProducts[i].id) {
          // console.log('false')
          return i
        }
      }
      return -1
    },
    /**
     * 合計金額の計算
     */
    totalPrice: function() {
      // console.log(this.putProducts.length)
      this.total = 0
      for(var i = 0; i < this.putProducts.length; i++) {
        this.total += this.putProducts[i].price * this.putProducts[i].amount
      }
      this.totalIncludedTax = this.total + (this.total * TAX_RATE)
    },
    move: function() {
      var clone = productElement[1].cloneNode(true)
      // clone.style.display = 'inline'
      clone.style.position = 'absolute'
      clone.style.top = '300px'
      purchasePageElement.appendChild(clone)
      console.log(productElement[1])
      console.log(clone)
      // product[1].style.position = 'absolute'
      // product[1].style.left = '300px'
    }
  }
}
</script>

<style>
/*
    親要素
*/
.purchasePage {
  display: flex;
}

/*
    商品一覧
*/
.product {
  width: 550px;
  height: 800px;
  border: solid 3px;
}

.productList {
  list-style: none;
}

.productList li {
  user-select: none;
}

/*
    カート
*/
.cart {
  position: relative;
  width: 550px;
  height: 800px;
  margin-left: 30px;
  border: solid 3px;
}

.putProductList {
  list-style: none;
}

.total {
  position: absolute;
  bottom: 0;
}
</style>
<template>
  <div class="purchasePage" @mouseup="reset">

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
        <li ref="product" v-for="product in products" :key="product.id" @mousedown="dragStart(product)">
          <div v-if="product.subject==subject">
            {{ product.id }} - {{ product.name }} - {{ product.price }}円
          </div>
        </li>
      </ul>
      <div ref="aaa">aaa</div>
      <button @click="move()">move</button>
    </div>

    <div class="cart" @mouseup="dragEnd">
      <ul class="putProductList">
        <li v-for="putProduct in putProducts" :key="putProduct.id">
          {{ putProduct.id }} - {{ putProduct.name }} - {{ putProduct.price }}円
        </li>
      </ul>
      <div class="total">
        合計: {{ total }}円
      </div>
    </div>
  </div>
</template>

<script>
// 選択中の科目
var selectedSubject = '国語';

// 商品の要素
var product = null

// ドラッグしているかどうか
var isDrag = false

export default {
  data() {
    return {
      // 商品一覧
      products: [
        {id: '0', subject: '国語', name: '国語の教科書', price: 2400},
        {id: '1', subject: '国語', name: '国語のドリル', price: 1500},
        {id: '2', subject: '国語', name: '国語のノート', price: 700},
        {id: '3', subject: '算数', name: '算数の教科書', price: 2600},
        {id: '4', subject: '算数', name: '算数のドリル', price: 1800},
        {id: '5', subject: '算数', name: '算数のドリル2', price: 1600},
        {id: '6', subject: '算数', name: '算数のノート', price: 600},
        {id: '7', subject: '英語', name: '英語の教科書', price: 1200},
        {id: '8', subject: '英語', name: '英語のドリル', price: 1100},
        {id: '9', subject: '英語', name: '英語のノート', price: 700},
        {id: '10', subject: '理科', name: '理科の教科書', price: 3000},
        {id: '11', subject: '理科', name: '理科のノート', price: 900},
        {id: '12', subject: '社会', name: '社会の教科書', price: 2800},
        {id: '13', subject: '社会', name: '社会のノート', price: 800}
      ],
      // 表示中の科目
      subject: selectedSubject,

      // カート内商品
      putProducts: [],

      // 合計金額
      total: 0
    }
  },
  mounted: function() {
    // 商品の要素取得
    product = this.$refs.product
  },
  methods: {
    /**
     * ドラッグ開始
     */
    dragStart: function(targetProduct) {
      console.log('ドラッグ開始')
      isDrag = true
      product = targetProduct
    },
    /**
     * ドラッグ中
     */
    dragging: function(product) {

    },
    /**
     * ドラッグ終了
     */
    dragEnd: function() {
      if(!isDrag) {
        return
      }
      console.log('ドラッグ終了')
      isDrag = false
      // console.log(this.putProducts.length)
      this.putProducts[this.putProducts.length] =
        { id: product.id, subject: product.subject, name: product.name, price: product.price }
      this.total += product.price
    },
    reset: function() {
      product = null
      isDrag = false
    },
    // move: function() {
    //   product[1].style.position = 'absolute'
    //   product[1].style.left = '300px'

    //   var aaa = this.$refs.aaa
    //   aaa.style.position = 'absolute'
    //   console.log(product[1].style)
    //   aaa.style.top = '100px'
    //   aaa.style.left = '100px'
    // }
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
  width: 500px;
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
  width: 500px;
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
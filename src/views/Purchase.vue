<template>
  <div ref="purchasePage" class="purchasePage" @mousemove="dragging" @mouseup="reset">
    <!-- 商品一覧 -->
    <div class="product">
      <h2>商品一覧</h2>
      <select v-model="subject">
        <option>国語</option>
        <option>算数</option>
        <option>英語</option>
        <option>理科</option>
        <option>社会</option>
      </select>
      {{ products.subject }}
      <table class="productList">
        <thead>
          <tr>
            <th>商品ID</th>
            <th>商品名</th>
            <th>値段(税抜)</th>
          </tr>
        </thead>
        <tbody v-for="product in products" :key="product.id">
          <tr v-if="product.subject==subject" @mousedown="dragStart(product, $event)">
            <td>{{ product.id }}</td>
            <td ref="product">{{ product.name }}</td>
            <td>{{ product.price }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <!-- カート -->
    <div class="cart" @mouseup="dragEnd">
      <h2>カート</h2>
      <table class="putProductList">
        <thead>
          <tr>
            <th>商品ID</th>
            <th>商品名</th>
            <th>値段(税抜)</th>
            <th>個数</th>
            <th>削除</th>
          </tr>
        </thead>
        <tbody v-for="(putProduct, index) in putProducts" :key="putProduct.id">
          <tr>
            <td class="putProductId">{{ putProduct.id }}</td>
            <td class="putProductName">{{ putProduct.name }}</td>
            <td class="putProductPrice">{{ putProduct.price }}</td>
            <td class="putProductAmount">
              <input type="text" :value="putProduct.amount"
                @input="amountValidation($event.target.value, index);totalPrice()"
                @change="replaceOne($event.target.value, index)"/>
              <div class="increaseOrDecrease">
                <button @click="changeAmount('+', index)">+</button>
                <button @click="changeAmount('-', index)">-</button>
              </div>
            </td>
            <td class="putProductDelete" @click="deleteProduct(index)"><button>X</button></td>
          </tr>
        </tbody>
      </table>
      <div class="total">
        合計: {{ total }}(税込: {{ totalIncludedTax }})円
      </div>
      <td class="putProductDeleteAll" @click="deleteProductAll()"><button>カートをクリア</button></td>
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
const TAX_RATE = 0.1
// 最大および最小購入数
const MAX_AMOUNT = 99
const MIN_AMOUNT = 1

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
      // console.log(product.sort_num)
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
      // console.log('ドラッグ終了')
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
        if(id === this.putProducts[i].id) {
          return i
        }
      }
      return -1
    },
    /**
     * 合計金額の計算
     */
    totalPrice: function() {
      this.total = 0
      for(var i = 0; i < this.putProducts.length; i++) {
        this.total += this.putProducts[i].price * this.putProducts[i].amount
      }
      this.totalIncludedTax = this.total + (this.total * TAX_RATE)
    },
    amountValidation: function(amount, index) {
      // フォーカス中は空文字を許可
      if(amount === '') {
        this.$set(this.putProducts, index, {
                                            id: this.putProducts[index].id,
                                            subject: this.putProducts[index].subject,
                                            name: this.putProducts[index].name,
                                            price: this.putProducts[index].price,
                                            amount: ''
                                          })
        return
      }

      // 購入範囲外の入力は受け付けない
      if(amount < MIN_AMOUNT || amount > MAX_AMOUNT) {
        return
      }

      // 自然数以外は「1」に置き換える
      if(amount.match('^[1-9][0-9]*$') != null) {
        this.$set(this.putProducts, index, {
                                            id: this.putProducts[index].id,
                                            subject: this.putProducts[index].subject,
                                            name: this.putProducts[index].name,
                                            price: this.putProducts[index].price,
                                            amount: amount
                                          })
      } else {
        this.$set(this.putProducts, index, {
                                            id: this.putProducts[index].id,
                                            subject: this.putProducts[index].subject,
                                            name: this.putProducts[index].name,
                                            price: this.putProducts[index].price,
                                            amount: 1
                                          })
      }
    },
    /**
     * 購入数変更
     */
    changeAmount: function(input, index) {
      switch(input) {
        case '+':
          if(this.putProducts[index].amount < MAX_AMOUNT) {
            this.$set(this.putProducts, index, {
                                                id: this.putProducts[index].id,
                                                subject: this.putProducts[index].subject,
                                                name: this.putProducts[index].name,
                                                price: this.putProducts[index].price,
                                                amount: parseInt(this.putProducts[index].amount) + 1
                                              })
          }
          break
        case '-':
          if(this.putProducts[index].amount > MIN_AMOUNT) {
            this.$set(this.putProducts, index, {
                                              id: this.putProducts[index].id,
                                              subject: this.putProducts[index].subject,
                                              name: this.putProducts[index].name,
                                              price: this.putProducts[index].price,
                                              amount: parseInt(this.putProducts[index].amount) - 1
                                            })
          }
          break
      }
      //合計金額算出
      this.totalPrice()
    },
    /**
     * 空文字を「1」に置き換える
     */
    replaceOne: function(amount, index) {
      if(amount === '') {
        this.$set(this.putProducts, index, {
                                            id: this.putProducts[index].id,
                                            subject: this.putProducts[index].subject,
                                            name: this.putProducts[index].name,
                                            price: this.putProducts[index].price,
                                            amount: 1
                                          })
      }
    },
    /**
     * カート内商品削除（1商品ずつ）
     */
    deleteProduct: function(index) {
      this.putProducts.splice(index, 1)
      //合計金額算出
      this.totalPrice()
    },
    /**
     * カート内商品削除（全て）
     */
    deleteProductAll: function() {
      this.putProducts.splice(0)
      //合計金額算出
      this.totalPrice()
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
  width: 500px;
  height: 800px;
  border: solid 3px;
}

.productList {
  list-style: none;
  user-select: none;
  margin-left: 5px;
  border: solid 2px;
  border-collapse: collapse;
}

.productList th, .productList td {
  width: 158px;
  border: solid 2px;
}

.productList td {
  text-align: center;
  border: solid 2px;
}

/*
    カート
*/
.cart {
  position: relative;
  width: 630px;
  height: 800px;
  margin-left: 30px;
  border: solid 3px;
}

.putProductList {
  list-style: none;
  margin-left: 5px;
  border: solid 2px;
  border-collapse: collapse;
}

.putProductList th {
  width: 145px;
  border: solid 2px;
}

.putProductList td {
  text-align: center;
}

.putProductId, .putProductName, .putProductPrice, .putProductDelete{
  border: solid 2px;
}

.putProductAmount {
  display: flex;
  box-shadow : 0 0 0 2px;
}

.increaseOrDecrease {
  display: flex;
  flex-direction: column;
}

.increaseOrDecrease button {
  width: 20px;
  height: 20px;
}

.total {
  position: absolute;
  bottom: 0;
}
</style>
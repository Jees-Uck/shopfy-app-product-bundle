<template>
  <div id="app">
    <div v-if="products" class="bundle-container">
      <p class="b-heading">
        <span class="b-heading__title">Create your Bundle </span><span v-if="discount" class="b-heading__discount"> and save {{ discount }}% </span><span class="b-heading__selecting">({{ selecting.length }}/{{ maxItems }}  Selected)</span></p>
      <div v-if="discount">
        <div class="total">Total: <span class="new-price">{{ ((total - total*discount/100)/100).toFixed(2) }} {{ currency }}</span>
          <span v-if="total" class="old-price">{{ (total / 100).toFixed(2) }} {{ currency }}</span></div>
      </div>
      <div v-else class="total">Total: {{ (total / 100).toFixed(2) }} {{ currency }}</div>
      <ProductList
          :products="products"
          :currency="currency"
          :add-disable="addDisable"
          @add-product="addProduct"
          @remove-product="removeProduct"
      ></ProductList>
      <div class="selected-products">
        <p v-show="resultList.length" class="selected-title">Selected Products:</p>
        <p v-for="item in resultList"
           :key="item.id"
           class="list-item">
         <span>{{ item.title }} X{{ item.quantity }}</span>
        </p>
        <transition name="slide-fade">
          <div v-show="message.visible" class="cart-message">{{ message.text }}</div>
        </transition>
      </div>
      <button :disabled="cartDisable" @click.prevent="addToCart" class="cart-add" id="add-bundle">Add to Cart</button>
      <button :disabled="buyNowDisable" @click.prevent="moveToCheckout" class="buy-now" id="buy-now">Buy Now</button>
    </div>
  </div>
</template>

<script>

import ProductList from "./components/ProductList";
import axios from "axios";

export default {
  name: 'App',
  data() {
    return {
      handles: [],
      response_products: [],
      products: [],
      selecting: [],
      resultList: [],
      cartItems: "",
      maxItems: 0,
      quantity: 0,
      total: 0,
      sellingPlan: "",
      discount: 0,
      currency: "",
      message: {
        visible: false,
        text: "Bundle added to your cart. "
      },
      checkoutLink: "",
      host: "https://test.web-space.com.ua/",
    }
  },
  components: {
    ProductList
  },
  methods: {
    addProduct(product) {
      this.selecting.push(product)
      product.quantity++
      let index = this.resultList.findIndex(item => item.id === product.id)
      if (index === -1) {
        this.resultList.push(product)
      }
      this.total += product.price
    },
    async removeProduct(id) {
      let index = this.selecting.findIndex(item => item.id === id)
      this.total -= this.selecting[index].price
      this.selecting[index].quantity--
      this.selecting.splice(index, 1)
      let idx = this.resultList.findIndex(item => item.id === id)
      if (!this.resultList[idx].quantity) {
        this.resultList.splice(idx, 1)
      }
    },
    getAppDataFromHtml() {
      const handleList = document.querySelectorAll('.handle-list__item');
      for (let i = 0; i < handleList.length; i++) {
        this.handles.push(handleList[i].innerText);
      }
      this.maxItems = document.getElementById('max-items').innerText;
      this.discount = document.getElementById('discount-value').innerText;
      this.currency = document.getElementById('currency').innerText;
    },
    async getProducts() {
      for (let i = 0; i < this.handles.length; i++) {
        //let response = await axios.get(this.host + this.handles[i])
        let response = await axios.get(this.handles[i] + '.js') //in shopify
        let data = response.data
        this.response_products.push(data)
      }
      console.table(this.response_products)
    },
    copyProductsWithQuantity() {
      this.products = this.response_products.map(function (product) {
        if (product.variants?.[0].available) {
          return {
            id: product.variants[0].id,
            title: product.title,
            price: product.variants[0].price,
            handle: product.handle,
            featured_image: product.featured_image,
            quantity: 0
          }
        }
      })
    },
    updateCartCounter() {
      axios.get('/cart.js')
          .then(response => {
            return response
          })
          .then(cart => {
            const counter = document.getElementById('CartToggleItemCount');
            cart = cart.data
            if (cart.item_count) {
              counter.innerHTML = cart.item_count;
              counter.classList.remove('hidden');
            }
          })
    },
    showCartMessage() {
      this.message.visible = true;
    },
    hideCartMessage() {
      setTimeout(() => {
        this.message.visible = false
        console.log(this.message)
      }, 4000)
    },
    async addToCart() {
      this.cartItems = this.resultList.map(function (product) {
        return {
          id: product.id,
          price: product.price,
          quantity: product.quantity,
        };
      });
      let items = this.cartItems;
      let url = '/cart/add.js'
      await axios.post(url, {
        headers: {
          'Content-Type': 'application/json',
        },
        items
      })

      this.updateCartCounter()
      this.copyProductsWithQuantity()
      this.selecting = []
      this.resultList = []
      this.total = 0
      await this.showCartMessage()
      this.hideCartMessage()
    },
    createCheckoutLink () {
      let products = this.resultList.map(function (product) {
        return product.id + ':' + product.quantity
      })
      this.checkoutLink = '/cart/' + products + '?discount=Bundle_Discount_' + this.discount
    },
    async moveToCheckout () {
      await this.createCheckoutLink()
      window.location.href = this.checkoutLink
    }
  },

  created() {
    this.getAppDataFromHtml()
  },
  async mounted() {
    await this.getProducts()
    await this.copyProductsWithQuantity()
  },
  computed: {
    addDisable() {
      return this.selecting.length >= this.maxItems
    },
    cartDisable() {
      return this.selecting.length < this.maxItems
    },
    buyNowDisable() {
      return this.selecting.length < this.maxItems
    }
  }
}
</script>

<style>
#app {
}
.bundle-container {
  max-width: 100%;
  min-width: 300px;
  margin: 0 auto;
  flex-direction: column;
  display: flex;
  justify-content: flex-start;
}
.selected-products {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
  margin-bottom: 25px;
  min-height: 25px;
}
.cart-add {
  display: inline;
  color: #FFF;
  background-color: #734A9E;
  padding: 15px;
  border-width: 0;
  border-radius: 30px;
  margin-bottom: 30px;
  cursor: pointer;
}
.cart-add[disabled] {
  opacity: .3;
}
.total {
  margin-bottom: 50px;
  font-weight: bold;
  color: #734A9E;
}
.b-heading {
  margin-bottom: 15px;
}
.selected-title {
  font-weight: bold;
}
.old-price {
  text-decoration: line-through;
}
.new-price {
  margin-right: 10px;
}
.cart-message {
  position: absolute;
  width: 100%;
}
.slide-fade-enter-active, .slide-fade-leave-active {
  transition: all .3s;
}
.slide-fade-enter, .slide-fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
  height: 0;
}
.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
  height: 0;
}

.slide-fade-leave-from,
.slide-fade-enter-to {
  opacity: 1;
  height: auto;
}
.carousel {
  width: 100%;
}
.b-heading__title, .b-heading__discount {
}
</style>

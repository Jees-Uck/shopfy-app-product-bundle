<template>
  <div id="app">
    <div class="total">Total {{ (total/100).toFixed(2) }}</div>
    <select class="selling-plan">
    </select>
    <div v-if="products" class="bundle-container">
      <ProductList
          :products="products"
          :add-disable="addDisable"
          @add-product="addProduct"
          @remove-product="removeProduct"
      ></ProductList>
      <p>Create your Bundle <span>({{ selecting.length }}/{{ maxItems }}  Selected)</span></p>
      <div class="selected-products">
        <p v-for="item in resultList"
           :key="item.id"
           class="list-item">{{ item.title }} X{{ item.quantity }}</p>
      </div>
      <button :disabled="cartDisable" @click.prevent="addToCart" class="cart-add">Add to Cart</button>
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
      addDisable: false,
      total: 0,
      sellingPlan: "",
      host: "https://test.web-space.com.ua/",
    }
  },
  components: {
    ProductList
  },
  methods: {
    setAddButtonVal() {
      this.addDisable = this.selecting.length >= this.maxItems;
    },
    addProduct(product) {
      this.selecting.push(product)
      product.quantity++
      this.setAddButtonVal()
      let index = this.resultList.findIndex(item => item.id === product.id)
      if (index === -1) {
        this.resultList.push(product)
      }
      this.total += product.price
      console.log(this.total)
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
      this.setAddButtonVal();
    },
    getMaxItems(){
      this.maxItems = document.getElementById('max-items').innerText;
    },
    getHandles() {
      const handleList = document.querySelectorAll('.handle-list__item');
      for (let i = 0; i < handleList.length; i++) {
        this.handles.push(handleList[i].innerText);
      }
    },
    async getProducts() {
       for (let i = 0; i < this.handles.length; i++) {
         let response = axios.get(this.host + this.handles[i])
         //let response = axios.get(this.handles[i] + '.js') //in shopify
         let data = (await response).data
         this.response_products.push(data)
      }
      console.table(this.response_products)
    },
    copyProductsWithQuantity() {
      this.products = this.response_products.map(function (product) {
        return {
          id: product.id,
          title: product.title,
          price: product.price,
          handle: product.handle,
          featured_image: product.featured_image,
          selling_plan: product.selling_plan,
          quantity: 0
        }
      })
    },
    addToCart() {
      this.cartItems = this.resultList.map(function (product) {
        return JSON.stringify({
          id: product.id,
          price: product.price,
          quantity: product.quantity,
          selling_plan: product.selling_plan,
        });
      });
      // let cartString = "{items: [" +`${this.cartItems}}`+"]";
      // axios.post('/cart.js', cartString)
      console.log(this.cartItems);
      // console.log(cartString);
    },
  },
  created() {
    this.getMaxItems();
  },
  async mounted() {
    await this.getHandles();
    await this.getProducts();
    await this.copyProductsWithQuantity();
  },
  computed: {
    cartDisable() {
      return this.selecting.length < this.maxItems
    }
  }
}
</script>

<style scoped>
#app {

}
.bundle-container {
  max-width: 100%;
  min-width: 300px;
  margin: 0 auto;
  flex-direction: column;
  display: flex;
  justify-content: center;
}
.selected-products {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  width: 100%;
  margin-bottom: 50px;
}
.cart-add {
  display: inline;
  color: #FFF;
  background-color: #734A9E;
  padding: 15px;
  max-width: 200px;
  border-width: 0px;
  border-radius: 30px;
  margin-bottom: 100px;
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
</style>

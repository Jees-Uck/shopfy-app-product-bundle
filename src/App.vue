<script setup>
import { ref, computed} from 'vue'
import { onMounted } from 'vue'
import axios from "axios";
import ProductList from "./components/ProductList";
import Spinner from "./components/Spinner";

const handles = []
const response_products = []
const products = ref([])
const selecting = ref([])
const resultList = ref([])
const total = ref(0)
const sellingPlan = ref('')
const messageVisible = ref(false)
const loading = ref(true)
const plansAreEqual = ref(true)
const discount = parseInt(document.getElementById('discount-value').innerText)
const maxItems = document.getElementById('max-items').innerText
const currency = document.getElementById('currency').innerText
const checkoutLink = ref('')
const host = 'https://test.web-space.com.ua/'

const addProduct = (product) => {
  selecting.value.push(product)
  product.quantity++
  let index = resultList.value.findIndex(item => item.id === product.id)
  if (index === -1) {
    resultList.value.push(product)
  }
  total.value += product.price
}

const removeProduct = (id) => {
  let index = selecting.value.findIndex(item => item.id === id)
  total.value -= selecting.value[index].price
  selecting.value[index].quantity--
  selecting.value.splice(index, 1)
  let resultIndex = resultList.value.findIndex(item => item.id === id)
  if (!resultList.value[resultIndex].quantity) {
    resultList.value.splice(resultIndex, 1)
  }
}

const createHandleList = () => {
  const handleItems = document.querySelectorAll('.handle-list__item');
  for (let i = 0; i < handleItems.length; i++) {
    handles.push(handleItems[i].innerText);
  }
}

const getProducts = async () => {
  for (let i = 0; i < handles.length; i++) {
    try {
      const response = await axios.get(host + handles[i]) // test
      //const response = await axios.get(handles[i] + '.js') //in Shopify
      let data = response.data
      response_products.push(data);
    } catch (err) {
      console.error('Product fetched with error: ', err)
    }
  }
}

const copyProductsWithQuantity = () => {
  products.value = response_products.map(product => {
    let sellingPlans = []

    product.variants[0].selling_plan_allocations.forEach(item => {
      sellingPlans.push(item.selling_plan_id.toString())
      sellingPlans.sort()
    })

    return {
      id: product.variants[0].id,
      title: product.title,
      price: product.variants[0].price,
      featured_image: product.featured_image,
      available: product.variants[0].available,
      sellingPlanIDs: sellingPlans,
      quantity: 0
    }
  })
}

const resetSelecting = () => {
  if (!plansAreEqual.value) {
    products.value.forEach(product => {
      product.quantity = 0
    })
    selecting.value = [];
    resultList.value = [];
    total.value = 0;
  }
}

const checkProductsPlans = () => {
  const plansToCompare = products.value.map((product) => {
    return {
      plans: product.sellingPlanIDs.join('')
    }
  })

  for (let i = 0; i < plansToCompare.length; i++) {
    if (plansToCompare[i].plans !== plansToCompare[i + 1].plans) {
      plansAreEqual.value = false
      return
    }
  }
}

const updateCartCounter = async () => {
  const counter = document.getElementById('CartToggleItemCount');
  try {
    const cartData = await axios.get('/cart.js');
    const cart = cartData.data;
    if (cart.item_count) {
      counter.innerHTML = cart.item_count;
      counter.classList.remove('hidden');
    }
  } catch (err) {
    console.error('Update counter error ', err)
  }
}

const setDiscount = () => {
  const discountCode = 'Bundle_Discount_' + discount;
  localStorage.setItem('discount', discountCode);
}

const displayMessage = () => {
  messageVisible.value = true;
  setTimeout(() => {
    messageVisible.value = false
  }, 4000)
}

const addToCart = async () => {
  try {
    let items = resultList.value.map(product => {
      return {
        id: product.id,
        price: product.price,
        quantity: product.quantity,
        selling_plan: this.sellingPlan
      };
    });
    let url = '/cart/add.js';
    //console.log('Cart items: ', items)
    //let url =  this.host + 'cart/add.js'; // test API
    await axios.post(url, {
      headers: {
        'Content-Type': 'application/json',
      },
      items
    })
    await updateCartCounter();
    if (!sellingPlan.value) {
      setDiscount();
    }
    displayMessage();
  } catch (err) {
    console.error('Add to cart error: ', err)
  }
  copyProductsWithQuantity();
  selecting.value = [];
  resultList.value = [];
  total.value = 0;
}

const createCheckoutLink = () => {
  let products = resultList.value.map(product => {
    return product.id + ':' + product.quantity
  })
  if (!sellingPlan.value) {
    checkoutLink.value = '/cart/' + products
        + '?discount=Bundle_Discount_' + discount
        + '&selling_plan=' + sellingPlan.value
  } else {
    checkoutLink.value = '/cart/' + products
        + '&selling_plan=' + sellingPlan.value
  }
}

const moveToCheckout = async () => {
  await createCheckoutLink();
  window.location.href = checkoutLink.value;
}

onMounted(async () => {
  await createHandleList()
  await getProducts()
  await copyProductsWithQuantity()
  loading.value = false
  checkProductsPlans()
})

const addIsDisable = computed(() => {
  return selecting.value.length >= maxItems
})

const cartIsDisable = computed(() => {
  return selecting.value.length < maxItems
})

const isOneTimePurchase = computed(() => {
  return !sellingPlan.value
})


const discountPrice = computed(() => {
  return ((total.value - total.value * discount / 100) / 100).toFixed(2)
})

</script>
<template>
    <input v-model="$i18n.locale"
           id="vue-lang"
           class="vue-lang"/>
    <input v-model="sellingPlan"
           id="selling-plan-vue"
           class="selling-plan"
           placeholder="Selling Plan"
           @input="resetSelecting">
    <!--    <input v-model="recurringPeriod" id="recurring-vue" placeholder="Recurring">-->
    <Spinner v-if="loading"></Spinner>
    <div v-else-if="products.length" class="bundle-container">
      <div class="b-heading">
        <div class="b-heading__title">
          {{ $t('title') }}
        </div>
        <div v-if="discount" class="b-heading__discount">
          {{ $t('discount_text') }}
          {{ discount }}%
        </div>
        <div class="b-heading__selecting">
          ({{ selecting.length }}/{{ maxItems }}
          {{ $t('selected_text') }})
        </div>
      </div>
      <div v-if="discount">
        <div class="total">{{ $t('total_text') }}:
          <div class="new-price">
            {{ discountPrice }}
            {{ currency }}
          </div>
          <div v-if="total" class="old-price">
            {{ (total / 100).toFixed(2) }}
            {{ currency }}
          </div>
        </div>
      </div>
      <div v-else class="total">
        {{ (total / 100).toFixed(2) }}
        {{ currency }}
      </div>
      <ProductList
          :products="products"
          :currency="currency"
          :add-disable="addIsDisable"
          :selling-plan="sellingPlan"
          @add-product="addProduct"
          @remove-product="removeProduct"
      ></ProductList>
      <div class="selected-products">
        <div v-show="resultList.length" class="selected-title">
          {{ $t('selected_products') }}
        </div>
        <div v-for="item in resultList"
             :key="item.id"
             class="list-item">
          <div>{{ item.title }} X{{ item.quantity }}</div>
        </div>
        <transition name="slide-fade">
          <div v-show="messageVisible" class="cart-message">
            {{ $t('cart_message') }}
          </div>
        </transition>
      </div>
      <button
          :disabled="cartIsDisable"
          @click.prevent="addToCart"
          class="cart-add"
          id="add-bundle">{{ $t('cart_text') }}
      </button>
      <button v-if="isOneTimePurchase"
              :disabled="cartIsDisable"
              @click.prevent="moveToCheckout"
              class="buy-now"
              id="buy-now">{{ $t('checkout_text') }}
      </button>
    </div>
    <div v-else>
      <p class="empty">
        {{ $t('empty_text') }}
      </p>
    </div>
</template>

<style>
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
  display: flex;
  gap: 5px;
  margin-bottom: 50px;
  font-weight: bold;
  color: #734A9E;
}
.b-heading {
  margin-bottom: 15px;
  display: flex;
  gap: 10px;
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
.slide-fade-enter, .slide-fade-leave-to {
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
.buy-now {
  background-color: #b2d436;
  border-color: #b2d436;
  border-width: 0;
  border-radius: 30px;
  margin-bottom: 30px;
  padding: 14px 50px;
  min-width: 250px;
  transition: opacity ease .2s;
  color: #FFFFFF;
}
button {
  cursor: pointer;
}
button:disabled, button[disabled] {
  opacity: .5;
}
</style>
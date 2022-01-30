<script setup>
import { defineProps, defineEmits } from 'vue'
import axios from 'axios'

defineProps({
  products: {
    type: Object,
    required: true,
  },
  purchaseIsDisabled: {
    type: Boolean,
    required: true,
  },
})
defineEmits()
const addToCart = async () => {
  try {
    let items = resultList.value.map(product => {
      return {
        id: product.id.value,
        price: product.price.value,
        quantity: product.quantity.value,
        selling_plan: sellingPlan.value,
      }
    })
    let url = '/cart/add.js'
    console.log('Cart items: ', items)
    //let url =  this.host + 'cart/add.js'; // test API
    await axios.post(url, {
      headers: {
        'Content-Type': 'application/json',
      },
      items,
    })
    await updateCartCounter()
    if (!sellingPlan.value) {
      setDiscount()
    }
    displayMessage()
  } catch (err) {
    console.error('Add to cart error: ', err)
  }
  copyProductsWithQuantity()
  selecting.value = []
  resultList.value = []
  total.value = 0
}
const setDiscount = () => {
  const discountCode = 'Bundle_Discount_' + discount
  if (oneTimePurchaseDiscount) {
    localStorage.setItem('discount', discountCode)
  }
}
const updateCartCounter = async () => {
  const counter = document.getElementById('CartToggleItemCount')
  try {
    const cartData = await axios.get('/cart.js')
    const cart = cartData.data
    if (cart.item_count) {
      counter.innerHTML = cart.item_count
      counter.classList.remove('hidden')
    }
  } catch (err) {
    console.error('Update counter error ', err)
  }
}
</script>
<template>
  <button :disabled="purchaseIsDisabled" @click.prevent="addToCart" class="cart-add" id="add-bundle">
    {{ $t('cart_text') }}
  </button>
</template>
<style scoped></style>

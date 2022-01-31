<script setup>
import { defineProps, defineEmits } from 'vue'
import axios from 'axios'

const props = defineProps({
  products: {
    type: Array,
    required: true,
  },
  purchaseIsDisable: {
    type: Boolean,
    required: true,
  },
  discount: {
    required: false,
  },
})
const emits = defineEmits(['cart-add'])
const cartAdd = message => {
  emits('cart-add', message)
}
const addToCart = async () => {
  try {
    let items = props.products.map(product => {
      return {
        id: product.id,
        price: product.price,
        quantity: product.quantity,
      }
    })
    let url = '/cart/add.js'
    console.log('Cart items: ', items, url)
    //let url =  this.host + 'cart/add.js'; // test API
    // await axios.post(url, {
    //   headers: {
    //     'Content-Type': 'application/json',
    //   },
    //   items,
    // })
    await updateCartCounter()
    await setDiscount()
    cartAdd('cart-add')
  } catch (err) {
    console.error('Add to cart error: ', err)
  }
}
const setDiscount = () => {
  const discountCode = 'Bundle_Discount_' + props.discount
  localStorage.setItem('discount', discountCode)
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
  <button :disabled="purchaseIsDisable" @click.prevent="addToCart('cart-add')" class="cart-add" id="add-bundle">
    {{ $t('cart_text') }}
  </button>
</template>
<style scoped></style>

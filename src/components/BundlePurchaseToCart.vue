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
        properties: { 'Discount': 'planted.set' }
      }
    })
    let url = '/cart/add.js'
    console.log('Cart items: ', items, url)
    //let url =  this.host + 'cart/add.js'; // test API
    await axios.post(url, {
      headers: {
        'Content-Type': 'application/json',
      },
      items,
    })
    await setDiscount()
    cartAdd('cart-add')
  } catch (err) {
    console.error('Add to cart error: ', err)
  }
}
const setDiscount = () => {
  if (props.discount) {
    const discountCode = 'Bundle_Discount_' + props.discount
    localStorage.setItem('discount', discountCode)
  }
}

</script>
<template>
  <button :disabled="purchaseIsDisable" @click.prevent="addToCart('cart-add')" class="buy-now" id="add-bundle">
    {{ $t('buy_now') }}
  </button>
</template>
<style scoped></style>

<script setup>
import { defineProps, ref } from 'vue'
import BundlePurchaseToCart from './BundlePurchaseToCart'
//import BundlePurchaseSubscription from './BundlePurchaseSubscription'
defineProps({
  purchaseIsDisabled: {
    type: Boolean,
    required: true,
  },
  products: {
    type: Array,
    required: true,
  },
  sellingPlan: {
    type: String,
    required: true,
  },
})
const messageVisible = ref(false)

const displayMessage = () => {
  messageVisible.value = true
  setTimeout(() => {
    messageVisible.value = false
  }, 4000)
}

</script>
<template>
  <div v-show="messageVisible" class="cart-message">
    {{ $t('cart_message') }}
  </div>
  <BundlePurchaseToCart :products="products" :cart-is-disable="purchaseIsDisabled" @cart-add="displayMessage" />
<!--  <BundlePurchaseSubscription :products="products" :cart-is-disable="purchaseIsDisabled" :selling-plan="sellingPlan" />-->
</template>
<style scoped></style>

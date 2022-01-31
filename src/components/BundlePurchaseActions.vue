<script setup>
import { defineProps, defineEmits } from 'vue'
import BundlePurchaseToCart from './BundlePurchaseToCart'
import BundlePurchaseSubscription from './BundlePurchaseSubscription'

defineProps({
  purchaseIsDisable: {
    type: Boolean,
    required: true,
  },
  subscription: {
    type: Object,
    required: true,
  },
  products: {
    type: Array,
    required: true,
  }
})
const emits = defineEmits(['cart-add', 'create-subscription'])

const createSubscription = (message) => {
  emits('create-subscription', message)
}
const cartAdd = (message) => {
  emits('cart-add', message)
}

</script>
<template>
  <div v-if="subscription.active">
    <BundlePurchaseSubscription
      :products="products"
      :purchase-is-disable="purchaseIsDisable"
      :subscription="subscription"
      @create-subscription="createSubscription"
    />
  </div>
  <div v-else>
    <BundlePurchaseToCart :products="products" :purchase-is-disable="purchaseIsDisable" @cart-add="cartAdd" />
  </div>
</template>
<style scoped></style>

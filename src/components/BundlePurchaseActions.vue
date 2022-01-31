<script setup>
import { defineProps, defineEmits, ref } from 'vue'
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
  },
  sellingPlan: {
    type: String,
    required: true,
  },
})
const emits = defineEmits(['create-subscription'])

const createSubscription = () => {
  emits('create-subscription')
}

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
  <div v-if="subscription.active">
    <BundlePurchaseSubscription
      :products="products"
      :purchase-is-disable="purchaseIsDisable"
      :subscription="subscription"
      @create-subscription="createSubscription"
    />
  </div>
  <div v-else>
    <BundlePurchaseToCart :products="products" :purchase-is-disable="purchaseIsDisable" @cart-add="displayMessage" />
  </div>
</template>
<style scoped></style>

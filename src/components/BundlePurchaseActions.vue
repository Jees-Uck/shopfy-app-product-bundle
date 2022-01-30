<script setup>
import { defineProps, ref } from 'vue'
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
const messageVisible = ref(false)

const displayMessage = () => {
  messageVisible.value = true
  setTimeout(() => {
    messageVisible.value = false
  }, 4000)
}
</script>
<template>
  {{ subscription }}
  <div v-show="messageVisible" class="cart-message">
    {{ $t('cart_message') }}
  </div>
  <transition-group name='fade'>
    <div v-if="subscription.active">
      Active Subscription
      <BundlePurchaseSubscription :products="products" :purchase-is-disable="purchaseIsDisable" />
    </div>
    <div v-else>
      <BundlePurchaseToCart
        :products="products"
        :purchase-is-disable="purchaseIsDisable"
        @cart-add="displayMessage"
      />
    </div>
  </transition-group>
</template>
<style scoped></style>

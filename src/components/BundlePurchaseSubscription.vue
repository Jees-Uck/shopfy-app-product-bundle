<script setup>
import { defineProps } from 'vue'

defineProps({
  products: {
    type: Array,
    required: true,
  },
  purchaseIsDisabled: {
    type: Boolean,
    required: true,
  },
  sellingPlan: {
    type: String,
    required: true,
  },
})
let checkoutLink = ''

const createCheckoutLink = () => {
  let productList = products.value.map(product => {
    return product.id + ':' + product.quantity
  })
  if (!sellingPlan) {
    checkoutLink = '/cart/' + productList + '?discount=Bundle_Discount_' + discount
    //+ '&selling_plan=' + sellingPlan.value
  } else {
    checkoutLink = '/cart/' + productList + '?selling_plan=' + sellingPlan.value
  }
}

const moveToCheckout = async () => {
  await createCheckoutLink()
  window.location.href = checkoutLink.value
}
</script>
<template>
  <button :disabled="purchaseIsDisabled" @click.prevent="moveToCheckout" class="buy-now" id="buy-now">
    {{ $t('checkout_text') }}
  </button>
</template>

<style scoped></style>
<!--Checkout action with Plan without discount in Shopify -->
<!--/cart/add?-->
<!--items[][id]={VARIANT_ID}&-->
<!--items[][quantity]={QUANTITY}&-->
<!--items[][selling_plan]={SELLING_PLAN_ID}&-->
<!--items[][id]={SECOND_VARIANT_ID}&-->
<!--items[][quantity]={SECOND_QUANTITY}&-->
<!--items[][selling_plan]={SELLING_PLAN_ID}&-->
<!--return_to=/checkout`-->

<!-- new link example-->
<!--/cart/add?id=40688353345722&quantity=&selling_plan=574030010&id=40688353378490&quantity=3&selling_plan=574030010&return_to=/checkout-->
<script setup>
import axios from 'axios'
import { defineProps, defineEmits } from 'vue'

const props = defineProps({
  products: {
    type: Array,
    required: true,
  },
  purchaseIsDisable: {
    type: Boolean,
    required: true,
  },
  subscription: {
    type: Object,
    required: true,
  },
})

const subscriptionEvent = message => {
  emits('create-subscription', message)
}

const emits = defineEmits(['create-subscription'])

const createSubscription = async () => {
  try {
    let items = props.products.map(product => {
      let index = product.sellingPlanIDs.indexOf(props.subscription.id)
      let price = product.priceAllocations[index].price
      return {
        id: product.id,
        price: price,
        quantity: product.quantity,
        selling_plan: props.subscription.id,
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
  } catch (err) {
    console.error('Add to cart error: ', err)
  }
}

// let checkoutLink = ''
//
// const createCheckoutLink = () => {
//   let productList = props.products.map(product => {
//     return 'id=' + product.id + '&quantity=' + product.quantity + '&selling_plan=' + props.subscription.id
//   })
//   checkoutLink = '/cart/add?' + productList + '&return_to=/checkout'
// }

const moveToCheckout = async () => {
  await createSubscription()
  subscriptionEvent('create-subscription')
  console.log()
  //window.location.href = '/checkout'
}
</script>
<template>
  <button :disabled="purchaseIsDisable" @click.prevent="moveToCheckout" class="buy-now" id="buy-now">
    {{ $t('subscribe_now') }}
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
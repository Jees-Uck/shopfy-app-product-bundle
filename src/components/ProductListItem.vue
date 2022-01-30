<script setup>
import 'vue3-carousel/dist/carousel.css'
import { Slide } from 'vue3-carousel'
import { defineProps, defineEmits, computed } from 'vue'

const props = defineProps({
  product: {
    type: Object,
    required: true,
  },
  addDisable: {
    type: Boolean,
    required: true,
  },
  currency: {
    type: String,
    required: false,
  },
  sellingPlan: {
    type: String,
    required: true,
  },
})
const emits = defineEmits(['add-product', 'remove-product'])

const addProduct = product => {
  emits('add-product', product)
}
const removeProduct = id => {
  emits('remove-product', id)
}
const removeIsDisable = computed(() => {
  return props.product.quantity <= 0
})

const productIsAvailable = computed(() => {
  return props.product.isInBundle && props.product.available
})
</script>
<template>
  <slide v-if="productIsAvailable" class="b-product">
    <div
      v-if="product.featured_image"
      class="b-product__image"
      :style="{ 'background-image': 'url(' + `${product.featured_image}` + ')' }"
    ></div>
    <div v-else class="no-image"></div>
    <div class="b-product__title">
      {{ product.title }}
    </div>
    <div class="b-product__price">{{ (product.price / 100).toFixed(2) }} {{ currency }}</div>
    <div class="b-button__group">
      <button @click="removeProduct(product.id)" class="b-product__minus" :disabled="removeIsDisable">-</button>
      <span class="b-product__counter">{{ product.quantity }}</span>
      <button @click="addProduct(product)" class="b-product__plus" :disabled="addDisable">+</button>
    </div>
  </slide>
</template>

<style>
.b-product {
  display: flex;
  flex-direction: column;
  gap: 10px;
  text-align: center;
  padding: 0 10px;
}
.b-product__image {
  width: 100%;
  display: block;
  min-height: 90px;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}
.b-button__group {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}
.b-product__plus,
.b-product__minus {
  width: 24px;
  height: 24px;
  border: 2px solid #734a9e;
  background-color: transparent;
  border-radius: 50%;
  padding: unset;
  color: #734a9e;
  line-height: 1px;
  margin: 0px;
  font-size: unset;
  transition: opacity ease 0.2s;
}
.b-product__counter {
  display: block;
  min-width: 30px;
}
.b-product__plus:disabled,
.b-product__minus:disabled {
  opacity: 0.3;
}
.b-product__price {
  color: #734a9e;
}
</style>

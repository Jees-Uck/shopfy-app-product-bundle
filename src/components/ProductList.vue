<script setup>
import { defineProps, defineEmits } from 'vue'
import ProductListItem from './ProductListItem'
import 'vue3-carousel/dist/carousel.css'
import { Carousel, Navigation } from 'vue3-carousel'

defineProps({
  products: {
    type: Array,
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
    required: false,
  },
})

const emits = defineEmits(['add-product', 'remove-product'])

const settings = {
  itemsToShow: 1,
  snapAlign: 'center',
}

const addProduct = product => {
  emits('add-product', product)
}

const removeProduct = id => {
  emits('remove-product', id)
}

const breakpoints = {
  500: {
    itemsToShow: 3,
    snapAlign: 'center',
  },
  768: {
    itemsToShow: 2,
    snapAlign: 'center',
  },
  940: {
    itemsToShow: 3,
    snapAlign: 'center',
  },
  1200: {
    itemsToShow: 4,
    snapAlign: 'start',
  },
  1400: {
    itemsToShow: 5,
    snapAlign: 'start',
  },
}
</script>
<template>
  <div id="bundle-list" class="b-product-list">
    <carousel :settings="settings" :breakpoints="breakpoints">
      <ProductListItem
        class="b-product"
        v-for="product in products"
        :add-disable="addDisable"
        :product="product"
        :key="product.id"
        :currency="currency"
        :selling-plan="sellingPlan"
        @add-product="addProduct"
        @remove-product="removeProduct"
      ></ProductListItem>
      <template #addons>
        <navigation />
      </template>
    </carousel>
  </div>
</template>
<style>
.b-product-list {
  display: flex;
  flex-direction: row;
  gap: 20px;
  width: 100%;
  margin-bottom: 30px;
}
.b-product-list .carousel {
  margin-right: -10px;
  margin-left: -10px;
}
</style>

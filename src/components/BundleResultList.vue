<script setup>
import { defineProps } from 'vue'
defineProps({
  products: {
    type: Array,
    required: true,
  },
})
</script>
<template>
  <div class="selected-products">
    <transition name="fade">
      <div v-if="products.length" class="selected-title">
        {{ $t('selected_products') }}
      </div>
    </transition>
    <transition-group name="fade">
      <div v-for="product in products" :key="product.id" class="list-item">
        <div>{{ product.title }} X{{ product.quantity }}</div>
      </div>
    </transition-group>
  </div>
</template>
<style scoped></style>

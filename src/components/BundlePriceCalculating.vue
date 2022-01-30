<script setup>
import { defineProps, computed } from 'vue'

const props = defineProps({
  settings: {
    type: Object,
    required: true,
  },
  subscription: {
    type: Object,
    required: true,
  },
  selecting: {
    type: Array,
    required: true,
  },
  total: {
    type: Number,
    required: true,
  },
})

const discount = computed(()=> {
  return props.subscription.active ? props.subscription.discount : props.settings.bundleDiscount
})

const discountPrice = computed(() => {
  return ((props.total - (props.total * discount.value) / 100) / 100).toFixed(2)
})
</script>
<template>
  <div class="b-heading">
    <div class="b-heading__title">
      {{ $t('title') }}
    </div>
    <div v-if="discount" class="b-heading__discount">
      {{ $t('discount_text') }}
      {{ discount }}%
    </div>
    <div class="b-heading__selecting">
      ({{ props.selecting.length }}/{{ props.settings.maxItems }} {{ $t('selected_text') }})
    </div>
  </div>
  <div v-if="discount">
    <div class="total">
      {{ $t('total_text') }}:
      <div class="new-price">
        {{ discountPrice }}
        {{ props.settings.currency }}
      </div>
      <div v-if="total" class="old-price">
        {{ (props.total / 100).toFixed(2) }}
        {{ props.settings.currency }}
      </div>
    </div>
  </div>
  <div v-else class="total">
    {{ (props.total / 100).toFixed(2) }}
    {{ props.settings.currency }}
  </div>
</template>
<style scoped></style>

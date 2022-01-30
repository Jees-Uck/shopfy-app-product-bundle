<script setup>
import { defineProps, defineEmits } from 'vue'

defineProps({
  mainProduct: {
    type: Object,
    required: true
  },
  modelValue: String
})
defineEmits(['update:modelValue'])
//
// const setPurchase = (value) => {
//   emits('select-purchase', value)
// }
</script>

<template>
<div class='b-purchase-options'>
  <div class='b-purchase__option'>
    <label for="onetime">
      <input type="radio" name="radio-input" value="onetime" checked
             id="onetime"
             @change="$emit('update:modelValue', 'onetime')" />
      One Time Purchase
    </label>
    <label for="subscription">
      <input type="radio" name="radio-input" value="subscription"
             id="subscription"
             @change="$emit('update:modelValue', 'subscription')" />
      Subscription
    </label>
    {{ isSubscription }}
  </div>
  <div class='b-purchase__option'>
  </div>
</div>
</template>

<style scoped>

</style>
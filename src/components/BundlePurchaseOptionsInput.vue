<script setup>
import { defineProps, defineEmits, ref } from 'vue'
const props = defineProps({
  plan: {
    type: Object,
    required: true,
  },
})
defineEmits(['update-subscription'])

let frequency = props.plan.options[0].value.split(' ')
let period = frequency[1]
</script>
<template>
  <label :for="plan.id" class="b-purchase__label">
    <input
      type="radio"
      @change="$emit('update-subscription', plan)"
      :id="plan.id"
      name="purchase"
      :value="plan.id"
      class="b-purchase__input"
    />
    {{ frequency[0] }} {{ $t(period) }} {{ $t('subscription') }}, {{ $t('save') }}
    {{ plan.price_adjustments[0].value }}%
  </label>
</template>

<style scoped></style>

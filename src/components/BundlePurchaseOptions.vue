<script setup>
import { defineProps, defineEmits } from 'vue'

defineProps({
  plans: {
    type: Array,
    required: true,
  },
  subscription: {
    type: Object,
    required: true,
  },
  //modelValue: String
})
defineEmits(['update-subscription'])
//defineEmits(['update:modelValue'])
//
// const setPurchase = value => {
//   emits('update-subscription', value)
// }
const onetime = {
  id: null,
}
</script>

<template>
  {{ subscription }}
  <div class="b-purchase-options">
    <div class="b-purchase__option">
      <ul>
        <li v-for="plan in plans" :key="plan.id">
          ID: {{ plan.id }} Name: {{ plan.name }} Discount: {{ plan.price_adjustments[0].value }}
        </li>
      </ul>
      <form>
        <label for="onetime">
          <input
            type="radio"
            name="purchase"
            value=""
            id="onetime"
            @change="$emit('update-subscription', onetime)"
            checked="checked"
          />
          One Time Purchase
        </label>
        <label v-for="plan in plans" :key="plan.id" :for="plan.id">
          <input
            type="radio"
            @change="$emit('update-subscription', plan)"
            :id="plan.id"
            name="purchase"
            :value="plan.id"
          />
          {{ plan.name }}
        </label>
      </form>
    </div>
    <div class="b-purchase__option"></div>
  </div>
</template>

<style scoped></style>

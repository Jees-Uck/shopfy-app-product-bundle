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
      <form class='b-purchase-options'>
        <label for="onetime">
          <input
            type="radio"
            name="purchase"
            value=""
            id="onetime"
            @change="$emit('update-subscription', onetime)"
            checked="checked"
          />
          {{ $t('one_time_purchase') }}
        </label>
        <label v-for="plan in plans" :key="plan.id" :for="plan.id">
          <input
            type="radio"
            @change="$emit('update-subscription', plan)"
            :id="plan.id"
            name="purchase"
            :value="plan.id"
          />
<!--          {{ $t('subscription_title') }}-->
          {{ plan.name }}
        </label>
      </form>
</template>

<style scoped>
 .b-purchase-options {
   display: flex;
   flex-direction: column;
 }
</style>

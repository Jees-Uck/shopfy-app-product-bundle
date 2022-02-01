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
  <form class="b-purchase-options">
    <label for="onetime" class='b-purchase__label'>
      <input
        type="radio"
        name="purchase"
        value=""
        id="onetime"
        @change="$emit('update-subscription', onetime)"
        checked="checked"
        class="b-purchase__input"
      />
      {{ $t('one_time_purchase') }}
    </label>
    <label v-for="plan in plans" :key="plan.id" :for="plan.id" class='b-purchase__label'>
      <input
        type="radio"
        @change="$emit('update-subscription', plan)"
        :id="plan.id"
        name="purchase"
        :value="plan.id"
        class="b-purchase__input"
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
.b-purchase__label {
  font-size: 16px;
}
.b-purchase__input {
  height: 15px;
  width: 15px;
  float: left;
  margin-right: 10px;
  margin-top: 3px;
  -webkit-appearance: auto;
  -moz-appearance: auto;
}
</style>

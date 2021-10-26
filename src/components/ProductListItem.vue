<template>
  <div class="b-product">
    <div v-if="product.featured_image" class="b-product__image" :style="{'background-image': 'url('+`${product.featured_image}`+')'}">
    </div>
    <div v-else class="no-image"></div>
    {{ product.title }}
    <div class="b-button__group">
      <button @click="$emit('add-product', product)"
              class="b-product__plus"
              :disabled="addDisable"
      >+</button>
     <span class="b-product__counter">{{ product.quantity }}</span>
      <button @click="$emit('remove-product', product.id)"
              class="b-product__minus"
              :disabled="rmDisable"
      >-</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "ProductListItem",
  props: {
    product: {
      type: Object,
      required: true
    },
    addDisable: {
      type: Boolean,
      required: true
    }
  },
  computed: {
    rmDisable () {
      return this.product.quantity <= 0
    }
  }
}
</script>

<style scoped>
.b-product {
  display: flex;
  flex-direction: column;
  gap: 10px;
  text-align: center;
}
.b-product__image {
 display: block;
  min-height: 90px;
  background-size: cover;
}
.b-button__group {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}
.b-product__plus, .b-product__minus {
  width: 24px;
  height: 24px;
  border: 2px solid #734A9E;
  background-color: transparent;
  border-radius: 50%;
  padding: unset;
  color: #734A9E;
  line-height: 1px;
  margin: 0px;
  font-size: unset;
  transition: opacity ease .2s;
}
.b-product__counter {
  display: block;
  min-width: 30px;
}
.b-product__plus:disabled, .b-product__minus:disabled {
  opacity: .3;
}
</style>
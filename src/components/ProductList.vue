<template>
  <div class="b-product-list">
  <ProductListItem class="b-product"
                   v-for="product in products"
                   :add-disable="addDisable"
                   :product="product"
                   :key="product.id"
                   @add-product="addProduct"
                   @remove-product="removeProduct"
  ></ProductListItem>
  </div>
</template>

<script>
import ProductListItem from "./ProductListItem";
export default {
  name: "ProductList",
  components: {ProductListItem},
  props: {
    products: {
      type: Object,
      required: true
    },
    addDisable: {
      type: Boolean,
      required: true
    }
  },
  methods: {
    addProduct(product) {
      this.$emit('add-product', product)
    },
    removeProduct(id){
      this.$emit('remove-product', id)
    }
  }
}
</script>

<style scoped>
.b-product-list {
  display: flex;
  flex-direction: row;
  gap: 20px;
  width: 100%;
  margin-bottom: 50px;
}

</style>
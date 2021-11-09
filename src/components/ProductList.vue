<template>
  <div id="bundle-list" class="b-product-list">
    <carousel :settings="settings" :breakpoints="breakpoints">
    <ProductListItem class="b-product"
                   v-for="product in products"
                   :add-disable="addDisable"
                   :product="product"
                   :key="product.id"
                   :currency="currency"
                   @add-product="addProduct"
                   @remove-product="removeProduct"
  ></ProductListItem>
      <template #addons>
        <navigation />
      </template>
    </carousel>
  </div>
</template>

<script>
import ProductListItem from "./ProductListItem";
import 'vue3-carousel/dist/carousel.css';
import { Carousel, Navigation } from 'vue3-carousel';
export default {
  name: "ProductList",
  components: {ProductListItem, Carousel, Navigation},
  props: {
    products: {
      type: Object,
      required: true
    },
    addDisable: {
      type: Boolean,
      required: true
    },
    currency: {
      type: String,
      required: false
    }
  },
  data() {
    return {
      settings: {
        itemsToShow: 1,
        snapAlign: 'center',
      },
      breakpoints: {
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
      },
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
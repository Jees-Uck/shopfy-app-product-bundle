<script setup>
import { ref, computed } from 'vue'
import { onMounted } from 'vue'
import axios from 'axios'
import Preloader from './components/Preloader'
import ProductList from './components/ProductList'
import BundlePriceCalculating from './components/BundlePriceCalculating'
import BundlePurchaseActions from './components/BundlePurchaseActions'
import BundleResultList from './components/BundleResultList'
import BundlePurchaseOptions from './components/BundlePurchaseOptions'

// Collect raw data
const handles = []
const rawProductsData = []
const bundleSettings = ref({
  bundleDiscount: parseInt(document.getElementById('discount-value').innerText),
  maxItems: document.getElementById('max-items').innerText,
  currency: document.getElementById('currency').innerText,
  subscriptionDiscount: '',
})

const createHandleList = () => {
  const handleItems = document.querySelectorAll('.handle-list__item')
  for (let i = 0; i < handleItems.length; i++) {
    handles.push(handleItems[i].innerText)
  }
}
const host = 'https://test.web-space.com.ua/'

const sellingPlans = ref({})

const getSellingPlans = async () => {
  try {
    const response = await axios.get(host + 'set')
    //const response = await axios.get(location.href + '.js') //in Shopify
    sellingPlans.value = response.data.selling_plan_groups[0].selling_plans
  } catch (err) {
    console.error('Selling plans fetched with error: ', err)
  }
}

const getProducts = async () => {
  for (let i = 0; i < handles.length; i++) {
    try {
      const response = await axios.get(host + handles[i]) // test
      //const response = await axios.get(handles[i] + '.js') //in Shopify
      let data = response.data
      rawProductsData.push(data)
    } catch (err) {
      console.error('Product fetched with error: ', err)
    }
  }
}

// Prepare app data
// Set general app variables
const selecting = ref([])
const resultList = ref([])
const sellingPlan = ref('')
const total = ref(null)
const loading = ref(true)
const products = ref([])

// Create product list from raw data
const copyProductsWithQuantity = () => {
  products.value = rawProductsData.map(product => {
    let sellingPlans = []

    product.variants[0].selling_plan_allocations.forEach(item => {
      sellingPlans.push(item.selling_plan_id)
      sellingPlans.sort()
    })

    return {
      id: product.variants[0].id,
      title: product.title,
      price: product.variants[0].price,
      featured_image: product.featured_image,
      available: product.variants[0].available,
      sellingPlanIDs: sellingPlans,
      isInBundle: true,
      quantity: 0,
    }
  })
}

const subscription = ref({
  active: false,
  id: null,
  name: '',
  discount: '',
})

const checkProductsPlans = () => {
  if (subscription.value.active) {
    products.value.forEach(product => {
      product.sellingPlanIDs.includes(subscription.value.id)
        ? (product.isInBundle = true)
        : (product.isInBundle = false)
    })
  } else {
    products.value.forEach(product => {
      product.isInBundle = true
    })
  }
}

const addProduct = product => {
  selecting.value.push(product)
  product.quantity++
  let index = resultList.value.findIndex(item => item.id === product.id)
  if (index === -1) {
    resultList.value.push(product)
  }
  total.value += product.price
}

const removeProduct = id => {
  let index = selecting.value.findIndex(item => item.id === id)
  total.value -= selecting.value[index].price
  selecting.value[index].quantity--
  selecting.value.splice(index, 1)
  let resultIndex = resultList.value.findIndex(item => item.id === id)
  if (!resultList.value[resultIndex].quantity) {
    resultList.value.splice(resultIndex, 1)
  }
}

const setPurchase = plan => {
  if (plan.id) {
    subscription.value.active = true
    subscription.value.id = plan.id
    subscription.value.name = plan.name
    subscription.value.discount = plan.price_adjustments[0].value
  } else {
    subscription.value.active = false
    subscription.value.id = null
    subscription.value.name = 'onetime'
    subscription.value.discount = bundleSettings.value.bundleDiscount
  }
  checkProductsPlans()
  resetSelecting()
}

const resetSelecting = () => {
  products.value.forEach(product => {
    product.quantity = 0
  })
  selecting.value = []
  resultList.value = []
  total.value = 0
}

onMounted(async () => {
  await createHandleList()
  await getSellingPlans()
  await getProducts()
  await copyProductsWithQuantity()
  await checkProductsPlans()
  loading.value = false
})

const addIsDisable = computed(() => {
  return selecting.value.length >= bundleSettings.value.maxItems
})

const purchaseIsDisable = computed(() => {
  return selecting.value.length < bundleSettings.value.maxItems
})
</script>
<template>
  <input v-model="$i18n.locale" id="vue-lang" class="vue-lang" />
  <Preloader v-if="loading"></Preloader>
  <div v-else-if="products.length" class="bundle-container">
    {{ subscription }}
    <BundlePurchaseOptions
      :plans="sellingPlans"
      :subscription="subscription"
      @update-subscription="setPurchase($event)"
    />
    <BundlePriceCalculating
      :settings="bundleSettings"
      :subscription="subscription"
      :total="total"
      :selecting="selecting"
    />
    <ProductList
      :products="products"
      :currency="bundleSettings.currency"
      :add-disable="addIsDisable"
      :selling-plan="sellingPlan"
      @add-product="addProduct"
      @remove-product="removeProduct"
    ></ProductList>
    <BundleResultList :products="resultList" />
    <BundlePurchaseActions :products="resultList" :subscription="subscription" @purchase="resetSelecting" :purchaseIsDisable='purchaseIsDisable'/>
  </div>
  <div v-else>
    <p class="empty">
      {{ $t('empty_text') }}
    </p>
  </div>
</template>

<style></style>

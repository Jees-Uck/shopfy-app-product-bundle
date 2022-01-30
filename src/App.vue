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

const mainProduct = ref({})

const getMainProduct = async () => {
  try {
    const response = await axios.get(host + 'set')
    //const response = await axios.get(location.href + '.js') //in Shopify
    mainProduct.value = response.data
  } catch (err) {
    console.error('Product fetched with error: ', err)
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
// Create product list from raw data
const products = ref([])

const copyProductsWithQuantity = () => {
  products.value = rawProductsData.map(product => {
    let sellingPlans = []

    product.variants[0].selling_plan_allocations.forEach(item => {
      sellingPlans.push(item.selling_plan_id.toString())
      sellingPlans.sort()
    })

    return {
      id: product.variants[0].id,
      title: product.title,
      price: product.variants[0].price,
      featured_image: product.featured_image,
      available: product.variants[0].available,
      sellingPlanIDs: sellingPlans,
      quantity: 0,
    }
  })
}

const setDefaultSellingPlan = () => {
  sellingPlan.value = mainProduct.value.selling_plan_groups[0].selling_plans[0].id
}

// Check if products selling plans and product set are equal
const checkProductsPlans = () => {
  const plansToCompare = products.value.map(product => {
    return {
      plans: product.sellingPlanIDs.join(''),
    }
  })

  for (let i = 0; i < plansToCompare.length; i++) {
    if (plansToCompare[i].plans !== plansToCompare[i + 1].plans) {
      plansAreEqual.value = false
      return
    }
  }
}

// Set general app variables

const selecting = ref([])
const resultList = ref([])
const sellingPlan = ref('')
const total = ref(null)
const loading = ref(true)
const plansAreEqual = ref(true)
const purchaseType = ref('onetime')
//const isSubscription = ref(false)
//const subscriptionTitle = ref('')

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

const resetSelecting = () => {
  if (!plansAreEqual.value) {
    products.value.forEach(product => {
      product.quantity = 0
    })
    selecting.value = []
    resultList.value = []
    total.value = 0
  }
}

onMounted(async () => {
  await createHandleList()
  await getMainProduct()
  await setDefaultSellingPlan()
  await getProducts()
  await copyProductsWithQuantity()
  await checkProductsPlans()
  loading.value = false
  console.log(products.value[0])
})

const isOneTimePurchase = computed(() => {
  return purchaseType.value === 'onetime'
})

const addIsDisable = computed(() => {
  return selecting.value.length >= bundleSettings.value.maxItems
})

// const purchaseIsDisabled = computed(() => {
//   return selecting.value.length < bundleSettings.value.maxItems
// })
</script>
<template>
  <input v-model="$i18n.locale" id="vue-lang" class="vue-lang" />
  <input
    v-model="sellingPlan"
    id="selling-plan-vue"
    class="selling-plan"
    placeholder="Selling Plan"
    @input="resetSelecting"
  />
  <Preloader v-if="loading"></Preloader>
  <div v-else-if="products.length" class="bundle-container">
    <BundlePurchaseOptions :main-product="mainProduct" v-model="purchaseType" />
    {{ purchaseType }}
    <br />
    Is One Time purchase: {{ isOneTimePurchase }}
    <BundlePriceCalculating :settings="bundleSettings" :subsription="false" :selecting="selecting" :total="total" />
    <ProductList
      :products="products"
      :currency="bundleSettings.currency"
      :add-disable="addIsDisable"
      :selling-plan="sellingPlan"
      @add-product="addProduct"
      @remove-product="removeProduct"
    ></ProductList>
        <BundleResultList :products="resultList" />
     <BundlePurchaseActions :cart-disable="purchaseIsDisabled" :selling-plan="sellingPlan" @purchase="resetSelecting" />
  </div>
  <div v-else>
    <p class="empty">
      {{ $t('empty_text') }}
    </p>
  </div>
</template>

<style></style>

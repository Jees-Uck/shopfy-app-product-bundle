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
})

const createHandleList = () => {
  const handleItems = document.querySelectorAll('.handle-list__item')
  for (let i = 0; i < handleItems.length; i++) {
    handles.push(handleItems[i].innerText)
  }
}
//const host = 'https://test.web-space.com.ua/'

const sellingPlans = ref({})

const getSellingPlans = async () => {
  try {
    //const response = await axios.get(host + 'set')
    const response = await axios.get(location.href + '.js') //in Shopify
    sellingPlans.value = response.data.selling_plan_groups[0].selling_plans
  } catch (err) {
    console.error('Selling plans fetched with error: ', err)
  }
}

const getProducts = async () => {
  for (let i = 0; i < handles.length; i++) {
    try {
      //const response = await axios.get(host + handles[i]) // test
      const response = await axios.get(handles[i] + '.js') //in Shopify
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
const total = ref(0)
const loading = ref(true)
const products = ref([])
const purchaseEvent = ref('')
const messageIsVisible = ref(false)

// Create product list from raw data
const createProductList = () => {
  products.value = rawProductsData.map(product => {
    let sellingPlans = []
    let priceAllocations = []
    product.variants[0].selling_plan_allocations.forEach(item => {
      let priceAllocation = {
        id: item.selling_plan_id,
        price: item.price,
      }
      sellingPlans.push(item.selling_plan_id)
      priceAllocations.push(priceAllocation)
    })

    return {
      id: product.variants[0].id,
      title: product.title,
      price: product.variants[0].price,
      featured_image: product.featured_image,
      available: product.variants[0].available,
      priceAllocations: priceAllocations,
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
    subscriptionEmptyState()
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

const subscriptionEmptyState = () => {
  subscription.value.active = false
  subscription.value.id = null
  subscription.value.name = 'onetime'
  subscription.value.discount = bundleSettings.value.bundleDiscount
}

const showMessage = message => {
  messageIsVisible.value = true
  setTimeout(() => {
    messageIsVisible.value = false
    purchaseEvent.value = ''
  }, 4000)
  console.log(message)
}

onMounted(async () => {
  await createHandleList()
  await getSellingPlans()
  await getProducts()
  await createProductList()
  await checkProductsPlans()
  loading.value = false
})

const messageVisible = ref(false)

const purchaseBundleAfterActions = async event => {
  purchaseEvent.value = event
  await resetSelecting()
  await createProductList()
  selecting.value = []
  resultList.value = []
  total.value = 0
  showMessage(event)
}

const addIsDisable = computed(() => {
  return selecting.value.length >= bundleSettings.value.maxItems
})

const purchaseIsDisable = computed(() => {
  return selecting.value.length < bundleSettings.value.maxItems
})

const messageStyle = computed(() => {
  return messageVisible ? 'visible' : 'hidden'
})
</script>
<template>
  <input v-model="$i18n.locale" id="vue-lang" class="vue-lang" />
  <Preloader v-if="loading"></Preloader>
  <div v-else-if="products.length" class="bundle-container">
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
    <div class="messages" :class='messageStyle'>
      <transition-group name="fade">
        <div v-if="purchaseEvent === 'cart-add'" class="cart-message">
          {{ $t('cart_message') }}
        </div>
        <div v-if="purchaseEvent === 'create-subscription'" class="cart-message">
          {{ $t('subscription_message') }}
        </div>
      </transition-group>
    </div>
    <BundlePurchaseActions
      :products="resultList"
      :subscription="subscription"
      :purchaseIsDisable="purchaseIsDisable"
      :discount="bundleSettings.bundleDiscount"
      @create-subscription="purchaseBundleAfterActions"
      @cart-add="purchaseBundleAfterActions"
    />
  </div>
  <div v-else>
    <p class="empty">
      {{ $t('empty_text') }}
    </p>
  </div>
</template>

<style>
.messages.visible {
  height: 60px;
}
.messages.hidden {
  height: 0;
  transition: height 0.2s;
}
.fade-enter-active {
  transition: all 0.2s ease 0.1s;
}
.fade-leave-active {
  transition: opacity 0s;
}
.fade-enter-from {
  opacity: 0;
}
</style>

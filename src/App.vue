<script setup>
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'
import AppHeader from './components/AppHeader.vue'
import CategoryIcon from './components/CategoryIcon.vue'
import CatalogSidebar from './components/CatalogSidebar.vue'
import ProductCard from './components/ProductCard.vue'
import ProductPage from './components/ProductPage.vue'
import InfoPage from './components/InfoPage.vue'
import CartDrawer from './components/CartDrawer.vue'
import OrderModal from './components/OrderModal.vue'
import EmptyState from './components/EmptyState.vue'
import Toast from './components/Toast.vue'
import logoUrl from './megatrend_logo.svg'

const products = ref([])
const isLoading = ref(true)
const loadError = ref(null)

const searchQuery = ref('')
const selectedCategory = ref('Усі')
const selectedBrands = ref([])
const selectedAttributes = ref({})
const priceMin = ref(0)
const priceMax = ref(100000)
const sortBy = ref('default')
const onlyDiscounted = ref(false)
const onlyInStock = ref(false)

const activeProduct = ref(null)
const activeInfoPage = ref(null)
const autoScrollReviews = ref(false)
const isCatalogMenuOpen = ref(false)
const isMobileFiltersOpen = ref(false)

const isCartOpen = ref(false)
const isCheckoutOpen = ref(false)

const appliedPromo = ref('')
const promoError = ref('')

const toastMessage = ref('')
const toastType = ref('success')
const toastVisible = ref(false)
let toastTimer = null

const desktopPosterUrl = import.meta.env.BASE_URL + 'posters/new-year-2027.svg'
const mobilePosterUrl = import.meta.env.BASE_URL + 'posters/new-year-2027-mobile.svg'

const categoryMeta = [
  { name: 'Усі', subtitle: 'Весь асортимент' },
  { name: 'Ноутбуки', subtitle: 'Ігрові та робочі' },
  { name: 'Смартфони', subtitle: 'Флагмани та 5G' },
  { name: 'Планшети', subtitle: 'Для навчання й арт' },
  { name: 'Аудіо', subtitle: 'Навушники й акустика' },
  { name: 'Смарт-годинники', subtitle: 'Спорт і здоров\'я' },
  { name: 'Монітори', subtitle: '4K та 165-180 Гц' },
  { name: 'Периферія', subtitle: 'Миші та клавіатури' },
  { name: 'Аксесуари', subtitle: 'Зарядки, УМБ, SSD' }
]

const cart = ref(loadInitialCart())

function loadInitialCart() {
  try {
    const saved = localStorage.getItem('megatrend_cart')
    return saved ? JSON.parse(saved) : []
  } catch (e) {
    return []
  }
}

watch(
  cart,
  (newCart) => {
    try {
      localStorage.setItem('megatrend_cart', JSON.stringify(newCart))
    } catch (e) {}
  },
  { deep: true }
)

watch(searchQuery, (val) => {
  if (val.trim()) {
    if (activeProduct.value || activeInfoPage.value) {
      activeProduct.value = null
      activeInfoPage.value = null
      updateUrlHash('')
    }
  }
})

onMounted(async () => {
  window.addEventListener('hashchange', syncRouteFromHash)
  await fetchProducts()
  syncRouteFromHash()
})

onUnmounted(() => {
  window.removeEventListener('hashchange', syncRouteFromHash)
})

function updateUrlHash(hash) {
  if (hash) {
    if (window.location.hash !== '#' + hash) {
      window.history.pushState(null, '', '#' + hash)
    }
  } else {
    if (window.location.hash) {
      window.history.pushState(null, '', window.location.pathname + window.location.search)
    }
  }
}

function syncRouteFromHash() {
  const raw = decodeURIComponent(window.location.hash.replace(/^#/, ''))
  if (!raw || raw === '/') {
    activeProduct.value = null
    activeInfoPage.value = null
    return
  }

  if (raw.startsWith('/product/')) {
    const idNum = Number(raw.replace('/product/', ''))
    const found = products.value.find((p) => p.id === idNum)
    if (found) {
      activeInfoPage.value = null
      activeProduct.value = found
      return
    }
  }

  if (raw.startsWith('/brand/')) {
    const brandName = raw.replace('/brand/', '')
    activeProduct.value = null
    activeInfoPage.value = null
    selectedCategory.value = 'Усі'
    selectedBrands.value = [brandName]
    selectedAttributes.value = {}
    syncPriceBoundsToCategory('Усі')
    return
  }

  if (raw.startsWith('/category/')) {
    const catName = raw.replace('/category/', '')
    activeProduct.value = null
    activeInfoPage.value = null
    selectedBrands.value = []
    selectedAttributes.value = {}
    selectedCategory.value = catName
    syncPriceBoundsToCategory(catName)
    return
  }

  if (raw.startsWith('/page/')) {
    const pageId = raw.replace('/page/', '')
    activeProduct.value = null
    activeInfoPage.value = pageId
    return
  }
}

function loadUserReviews() {
  try {
    const raw = localStorage.getItem('megatrend_product_reviews')
    return raw ? JSON.parse(raw) : {}
  } catch (e) {
    return {}
  }
}

function saveUserReviews(map) {
  try {
    localStorage.setItem('megatrend_product_reviews', JSON.stringify(map))
  } catch (e) {}
}

function recalculateProductRating(product) {
  const list = Array.isArray(product.reviews) ? product.reviews : []
  product.reviewsCount = list.length
  if (list.length > 0) {
    const avg = list.reduce((sum, r) => sum + Number(r.rating || 5), 0) / list.length
    product.rating = Number(avg.toFixed(1))
  }
}

async function fetchProducts() {
  isLoading.value = true
  loadError.value = null
  try {
    const response = await fetch(import.meta.env.BASE_URL + 'products.json')
    if (!response.ok) {
      throw new Error('Помилка завантаження каталогу')
    }
    const loaded = await response.json()
    const savedReviewsMap = loadUserReviews()
    loaded.forEach((p) => {
      const customList = savedReviewsMap[p.id]
      if (Array.isArray(customList) && customList.length > 0) {
        const baseReviews = Array.isArray(p.reviews) ? p.reviews : []
        p.reviews = [...customList, ...baseReviews]
        recalculateProductRating(p)
      }
    })
    products.value = loaded
    syncPriceBoundsToCategory(selectedCategory.value)
  } catch (err) {
    loadError.value = err.message || 'Не вдалося завантажити товари'
  } finally {
    isLoading.value = false
  }
}

const categoryScopedProducts = computed(() => {
  if (selectedCategory.value === 'Усі') {
    return products.value
  }
  return products.value.filter((p) => p.category === selectedCategory.value)
})

const priceBounds = computed(() => {
  const list = categoryScopedProducts.value
  if (list.length === 0) return { min: 0, max: 100000 }
  const prices = list.map((p) => p.price)
  return {
    min: Math.min(...prices),
    max: Math.max(...prices)
  }
})

function syncPriceBoundsToCategory(catName) {
  const list =
    catName === 'Усі'
      ? products.value
      : products.value.filter((p) => p.category === catName)
  if (list.length === 0) {
    priceMin.value = 0
    priceMax.value = 100000
    return
  }
  const prices = list.map((p) => p.price)
  priceMin.value = Math.min(...prices)
  priceMax.value = Math.max(...prices)
}

const categoriesWithCount = computed(() => {
  return categoryMeta.map((c) => {
    const count =
      c.name === 'Усі'
        ? products.value.length
        : products.value.filter((p) => p.category === c.name).length
    return {
      ...c,
      count
    }
  })
})

const allBrandsList = computed(() => {
  const counts = {}
  products.value.forEach((p) => {
    if (p.brand) {
      counts[p.brand] = (counts[p.brand] || 0) + 1
    }
  })
  return Object.keys(counts)
    .sort()
    .map((name) => ({ name, count: counts[name] }))
})

const sidebarBrandsList = computed(() => {
  const counts = {}
  categoryScopedProducts.value.forEach((p) => {
    if (p.brand) {
      counts[p.brand] = (counts[p.brand] || 0) + 1
    }
  })
  return Object.keys(counts)
    .sort()
    .map((name) => ({ name, count: counts[name] }))
})

const sidebarAttributeGroups = computed(() => {
  const groupsMap = {}
  categoryScopedProducts.value.forEach((p) => {
    if (!p.filterAttributes) return
    Object.entries(p.filterAttributes).forEach(([groupName, val]) => {
      if (!val) return
      if (!groupsMap[groupName]) {
        groupsMap[groupName] = {}
      }
      groupsMap[groupName][val] = (groupsMap[groupName][val] || 0) + 1
    })
  })

  return Object.entries(groupsMap).map(([name, valuesObj]) => {
    const options = Object.entries(valuesObj)
      .map(([value, count]) => ({ value, count }))
      .sort((a, b) => a.value.localeCompare(b.value, 'uk'))
    return { name, options }
  })
})

const filteredProducts = computed(() => {
  let list = [...categoryScopedProducts.value]

  if (selectedBrands.value.length > 0) {
    const lowerSet = selectedBrands.value.map((b) => b.toLowerCase())
    list = list.filter((p) => p.brand && lowerSet.includes(p.brand.toLowerCase()))
  }

  list = list.filter((p) => p.price >= priceMin.value && p.price <= priceMax.value)

  Object.entries(selectedAttributes.value).forEach(([groupName, checkedValues]) => {
    if (Array.isArray(checkedValues) && checkedValues.length > 0) {
      list = list.filter(
        (p) =>
          p.filterAttributes &&
          p.filterAttributes[groupName] &&
          checkedValues.includes(p.filterAttributes[groupName])
      )
    }
  })

  if (searchQuery.value.trim()) {
    const q = searchQuery.value.trim().toLowerCase()
    list = list.filter(
      (p) =>
        p.name.toLowerCase().includes(q) ||
        (p.brand && p.brand.toLowerCase().includes(q)) ||
        (p.sku && p.sku.toLowerCase().includes(q)) ||
        (p.fullDescription && p.fullDescription.toLowerCase().includes(q))
    )
  }

  if (onlyDiscounted.value) {
    list = list.filter((p) => p.oldPrice && p.oldPrice > p.price)
  }

  if (onlyInStock.value) {
    list = list.filter((p) => p.inStock)
  }

  if (sortBy.value === 'price-asc') {
    list.sort((a, b) => a.price - b.price)
  } else if (sortBy.value === 'price-desc') {
    list.sort((a, b) => b.price - a.price)
  } else if (sortBy.value === 'rating') {
    list.sort((a, b) => b.rating - a.rating)
  } else if (sortBy.value === 'name') {
    list.sort((a, b) => a.name.localeCompare(b.name, 'uk'))
  }

  return list
})

const activeAttributeChips = computed(() => {
  const chips = []
  Object.entries(selectedAttributes.value).forEach(([group, values]) => {
    if (Array.isArray(values)) {
      values.forEach((val) => {
        chips.push({ group, value: val })
      })
    }
  })
  return chips
})

const isPriceFilterCustom = computed(() => {
  return priceMin.value > priceBounds.value.min || priceMax.value < priceBounds.value.max
})

const activeFiltersCount = computed(() => {
  let c = 0
  if (selectedCategory.value !== 'Усі') c++
  c += selectedBrands.value.length
  c += activeAttributeChips.value.length
  if (isPriceFilterCustom.value) c++
  if (onlyDiscounted.value) c++
  if (onlyInStock.value) c++
  return c
})

const cartCount = computed(() => {
  return cart.value.reduce((total, item) => total + item.quantity, 0)
})

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => total + item.price * item.quantity, 0)
})

const discountAmount = computed(() => {
  if (!appliedPromo.value || cartTotal.value === 0) return 0
  const code = appliedPromo.value.toUpperCase()
  if (code === '2027') {
    return Math.round(cartTotal.value * 0.25)
  }
  if (code === 'STUDENT10' || code === 'KPI10') {
    return Math.round(cartTotal.value * 0.1)
  }
  if (code === 'VUE20') {
    return Math.round(cartTotal.value * 0.2)
  }
  if (code === 'PROMO500') {
    return Math.min(500, cartTotal.value)
  }
  return 0
})

const finalTotal = computed(() => {
  return Math.max(0, cartTotal.value - discountAmount.value)
})

function openProductPage(product) {
  autoScrollReviews.value = false
  activeInfoPage.value = null
  activeProduct.value = product
  isCatalogMenuOpen.value = false
  isMobileFiltersOpen.value = false
  updateUrlHash('/product/' + product.id)
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function openProductReviews(product) {
  autoScrollReviews.value = true
  activeInfoPage.value = null
  activeProduct.value = product
  isCatalogMenuOpen.value = false
  isMobileFiltersOpen.value = false
  updateUrlHash('/product/' + product.id)
}

function openInfoPage(pageId) {
  activeProduct.value = null
  activeInfoPage.value = pageId
  isCatalogMenuOpen.value = false
  isMobileFiltersOpen.value = false
  updateUrlHash('/page/' + pageId)
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function goHome() {
  activeProduct.value = null
  activeInfoPage.value = null
  selectedCategory.value = 'Усі'
  selectedBrands.value = []
  selectedAttributes.value = {}
  searchQuery.value = ''
  onlyDiscounted.value = false
  onlyInStock.value = false
  isCatalogMenuOpen.value = false
  isMobileFiltersOpen.value = false
  syncPriceBoundsToCategory('Усі')
  updateUrlHash('')
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function selectCategory(categoryName) {
  selectedCategory.value = categoryName
  selectedBrands.value = []
  selectedAttributes.value = {}
  activeProduct.value = null
  activeInfoPage.value = null
  isCatalogMenuOpen.value = false
  syncPriceBoundsToCategory(categoryName)
  if (categoryName === 'Усі') {
    updateUrlHash('')
  } else {
    updateUrlHash('/category/' + encodeURIComponent(categoryName))
  }
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function selectBrand(brandName) {
  selectedCategory.value = 'Усі'
  selectedBrands.value = [brandName]
  selectedAttributes.value = {}
  activeProduct.value = null
  activeInfoPage.value = null
  isCatalogMenuOpen.value = false
  syncPriceBoundsToCategory('Усі')
  updateUrlHash('/brand/' + encodeURIComponent(brandName))
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function toggleSidebarBrand(brandName) {
  const idx = selectedBrands.value.indexOf(brandName)
  if (idx >= 0) {
    selectedBrands.value.splice(idx, 1)
  } else {
    selectedBrands.value.push(brandName)
  }
}

function toggleSidebarAttribute({ group, value }) {
  const current = selectedAttributes.value[group] || []
  const idx = current.indexOf(value)
  if (idx >= 0) {
    const next = current.filter((v) => v !== value)
    if (next.length === 0) {
      const copy = { ...selectedAttributes.value }
      delete copy[group]
      selectedAttributes.value = copy
    } else {
      selectedAttributes.value = { ...selectedAttributes.value, [group]: next }
    }
  } else {
    selectedAttributes.value = { ...selectedAttributes.value, [group]: [...current, value] }
  }
}

function handleUpdatePrice({ min, max }) {
  priceMin.value = min
  priceMax.value = max
}

function resetPriceFilter() {
  priceMin.value = priceBounds.value.min
  priceMax.value = priceBounds.value.max
}

function applyPosterPromo() {
  appliedPromo.value = '2027'
  promoError.value = ''
  showToast('Новорічний промокод 2027 (-25%) активовано!')
}

function addToCart(product) {
  const existing = cart.value.find((item) => item.id === product.id)
  if (existing) {
    existing.quantity += 1
  } else {
    cart.value.push({
      id: product.id,
      name: product.name,
      category: product.category,
      price: product.price,
      image: product.image,
      quantity: 1
    })
  }
  showToast(`Додано в кошик: ${product.name}`)
}

function updateCartQuantity({ id, quantity }) {
  const item = cart.value.find((i) => i.id === id)
  if (item) {
    item.quantity = Math.max(1, quantity)
  }
}

function removeFromCart(id) {
  cart.value = cart.value.filter((i) => i.id !== id)
}

function clearCart() {
  cart.value = []
  appliedPromo.value = ''
  promoError.value = ''
}

function applyPromo(code) {
  promoError.value = ''
  const normalized = code.trim().toUpperCase()
  if (['2027', 'STUDENT10', 'KPI10', 'VUE20', 'PROMO500'].includes(normalized)) {
    appliedPromo.value = normalized
    const label = normalized === '2027' ? '2027 (-25%)' : normalized
    showToast(`Промокод ${label} застосовано`)
  } else {
    promoError.value = 'Введений промокод не знайдено'
  }
}

function removePromo() {
  appliedPromo.value = ''
  promoError.value = ''
}

function openCheckout() {
  isCartOpen.value = false
  isCheckoutOpen.value = true
}

function handleOrderPlaced(orderData) {
  cart.value = []
  appliedPromo.value = ''
  promoError.value = ''
  showToast(`Замовлення №${orderData.orderNumber} оформлено`)
}

function addProductReview({ productId, author, rating, text }) {
  const target = products.value.find((p) => p.id === productId)
  if (!target) return

  const now = new Date()
  const day = String(now.getDate()).padStart(2, '0')
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const year = now.getFullYear()
  const dateStr = `${day}.${month}.${year}`

  const newReview = {
    id: 'user-' + Date.now(),
    author,
    rating: Number(rating),
    date: dateStr,
    text: text || ''
  }

  if (!Array.isArray(target.reviews)) {
    target.reviews = []
  }
  target.reviews.unshift(newReview)
  recalculateProductRating(target)

  const savedMap = loadUserReviews()
  const existingCustom = Array.isArray(savedMap[productId]) ? savedMap[productId] : []
  savedMap[productId] = [newReview, ...existingCustom]
  saveUserReviews(savedMap)

  showToast('Дякуємо! Ваш відгук успішно додано')
}

function resetFilters() {
  searchQuery.value = ''
  selectedCategory.value = 'Усі'
  selectedBrands.value = []
  selectedAttributes.value = {}
  sortBy.value = 'default'
  onlyDiscounted.value = false
  onlyInStock.value = false
  syncPriceBoundsToCategory('Усі')
  updateUrlHash('')
}

function showToast(message, type = 'success') {
  toastMessage.value = message
  toastType.value = type
  toastVisible.value = true
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => {
    toastVisible.value = false
  }, 2600)
}
</script>

<template>
  <div class="page">
    <AppHeader
      v-model:search-query="searchQuery"
      :cart-count="cartCount"
      :cart-total="finalTotal"
      @toggle-cart="isCartOpen = !isCartOpen"
      @go-home="goHome"
      @toggle-catalog-menu="isCatalogMenuOpen = !isCatalogMenuOpen"
    />

    <Transition name="mega-dropdown">
      <div v-if="isCatalogMenuOpen" class="catalog-dropdown-overlay" @click="isCatalogMenuOpen = false">
        <div class="container catalog-modal-wrap" @click.stop>
          <div class="catalog-mega-menu">
            <div class="mega-top-bar">
              <h2 class="mega-top-title">Каталог</h2>
            </div>

            <div class="mega-body-grid">
              <div class="mega-cats">
                <h3 class="mega-heading">Категорії товарів</h3>
                <div class="mega-cats-grid">
                  <button
                    v-for="cat in categoriesWithCount"
                    :key="cat.name"
                    type="button"
                    class="mega-cat-item"
                    :class="{ active: selectedCategory === cat.name && selectedBrands.length === 0 }"
                    @click="selectCategory(cat.name)"
                  >
                    <span class="mega-cat-icon-box">
                      <CategoryIcon :name="cat.name" />
                    </span>
                    <div class="mega-cat-text">
                      <strong>{{ cat.name }}</strong>
                      <span>{{ cat.subtitle }} ({{ cat.count }})</span>
                    </div>
                  </button>
                </div>
              </div>

              <div class="mega-brands">
                <h3 class="mega-heading">Популярні бренди</h3>
                <div class="mega-brands-list">
                  <button
                    v-for="b in allBrandsList"
                    :key="b.name"
                    type="button"
                    class="mega-brand-chip"
                    :class="{ active: selectedBrands.includes(b.name) }"
                    @click="selectBrand(b.name)"
                  >
                    {{ b.name }} <span>({{ b.count }})</span>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </Transition>

    <main class="main container">
      <Transition name="page-fade" mode="out-in">
        <InfoPage
          v-if="activeInfoPage"
          :key="'info-' + activeInfoPage"
          :page="activeInfoPage"
          @back="goHome"
        />

        <ProductPage
          v-else-if="activeProduct"
          :key="'prod-' + activeProduct.id"
          :product="activeProduct"
          :all-products="products"
          :auto-scroll-reviews="autoScrollReviews"
          @back="goHome"
          @add-to-cart="addToCart"
          @open-product="openProductPage"
          @select-category="selectCategory"
          @select-brand="selectBrand"
          @add-review="addProductReview"
        />

        <div v-else key="catalog-main">
          <section class="poster-block" @click="applyPosterPromo" title="Натисніть, щоб активувати промокод 2027 (-25%)">
            <picture>
              <source media="(max-width: 680px)" :srcset="mobilePosterUrl" />
              <img
                :src="desktopPosterUrl"
                alt="Новорічний розпродаж 2027 - знижка 25% за промокодом 2027"
                class="poster-img"
              />
            </picture>
          </section>

          <section class="rozetka-categories">
            <button
              v-for="cat in categoriesWithCount"
              :key="cat.name"
              type="button"
              class="rz-cat-card"
              :class="{ active: selectedCategory === cat.name && selectedBrands.length === 0 }"
              @click="selectCategory(cat.name)"
            >
              <span class="rz-cat-icon-box">
                <CategoryIcon :name="cat.name" />
              </span>
              <div class="rz-cat-info">
                <span class="rz-cat-title">{{ cat.name }}</span>
                <span class="rz-cat-count">{{ cat.count }} товарів</span>
              </div>
            </button>
          </section>

          <div class="toolbar">
            <div class="active-filters">
              <button
                type="button"
                class="mobile-filters-btn"
                @click="isMobileFiltersOpen = true"
              >
                <span>⚙️ Фільтри</span>
                <span v-if="activeFiltersCount > 0" class="mobile-filters-badge">{{ activeFiltersCount }}</span>
              </button>

              <span class="found-count">
                Знайдено товарів: <strong>{{ filteredProducts.length }}</strong>
              </span>

              <span v-if="selectedCategory !== 'Усі'" class="filter-chip">
                Категорія: <strong>{{ selectedCategory }}</strong>
                <button type="button" @click="selectCategory('Усі')">✕</button>
              </span>

              <span v-if="isPriceFilterCustom" class="filter-chip">
                Ціна: <strong>{{ priceMin }} - {{ priceMax }} ₴</strong>
                <button type="button" @click="resetPriceFilter">✕</button>
              </span>

              <span
                v-for="bName in selectedBrands"
                :key="bName"
                class="filter-chip brand-chip"
              >
                Бренд: <strong>{{ bName }}</strong>
                <button type="button" @click="toggleSidebarBrand(bName)">✕</button>
              </span>

              <span
                v-for="chip in activeAttributeChips"
                :key="chip.group + '-' + chip.value"
                class="filter-chip"
              >
                {{ chip.group }}: <strong>{{ chip.value }}</strong>
                <button type="button" @click="toggleSidebarAttribute(chip)">✕</button>
              </span>

              <button
                v-if="activeFiltersCount > 0"
                type="button"
                class="clear-all-chips-btn"
                @click="resetFilters"
              >
                Скинути всі
              </button>
            </div>

            <div class="toolbar-right">
              <label class="checkbox-label">
                <input type="checkbox" v-model="onlyDiscounted" />
                <span>Тільки акційні</span>
              </label>

              <select v-model="sortBy" class="sort-select">
                <option value="default">За популярністю</option>
                <option value="price-asc">Від дешевих до дорогих</option>
                <option value="price-desc">Від дорогих до дешевих</option>
                <option value="rating">За рейтингом</option>
                <option value="name">За назвою (А-Я)</option>
              </select>
            </div>
          </div>

          <div v-if="isLoading" class="loading-state">
            Завантаження каталогу...
          </div>

          <div v-else-if="loadError" class="error-state">
            <p>{{ loadError }}</p>
            <button type="button" class="retry-btn" @click="fetchProducts">Спробувати ще раз</button>
          </div>

          <div v-else class="catalog-layout">
            <CatalogSidebar
              v-model:only-discounted="onlyDiscounted"
              v-model:only-in-stock="onlyInStock"
              :min-bound="priceBounds.min"
              :max-bound="priceBounds.max"
              :price-min="priceMin"
              :price-max="priceMax"
              :available-brands="sidebarBrandsList"
              :selected-brands="selectedBrands"
              :attribute-groups="sidebarAttributeGroups"
              :selected-attributes="selectedAttributes"
              :active-filters-count="activeFiltersCount"
              :filtered-count="filteredProducts.length"
              :is-mobile-open="isMobileFiltersOpen"
              @update-price="handleUpdatePrice"
              @toggle-brand="toggleSidebarBrand"
              @toggle-attribute="toggleSidebarAttribute"
              @reset-all="resetFilters"
              @close-mobile="isMobileFiltersOpen = false"
            />

            <div class="catalog-results">
              <EmptyState
                v-if="filteredProducts.length === 0"
                title="Товарів не знайдено"
                message="За обраними параметрами фільтрації товарів не знайдено. Спробуйте змінити критерії підбору."
                action-text="Скинути всі фільтри"
                @action="resetFilters"
              />

              <div v-else class="grid">
                <ProductCard
                  v-for="product in filteredProducts"
                  :key="product.id"
                  :product="product"
                  @add-to-cart="addToCart"
                  @open-product="openProductPage"
                  @open-reviews="openProductReviews"
                  @select-brand="selectBrand"
                />
              </div>
            </div>
          </div>
        </div>
      </Transition>
    </main>

    <footer class="footer">
      <div class="container">
        <div class="footer-top">
          <div class="footer-brand">
            <img :src="logoUrl" alt="МегаТренд" class="footer-logo" @click="goHome" />
            <p class="footer-desc">
              Інтернет-магазин сучасної комп'ютерної техніки, смартфонів, планшетів, моніторів, аудіо та смарт-гаджетів з офіційною гарантією.
            </p>
          </div>

          <div class="footer-col">
            <h4 class="footer-title">Каталог</h4>
            <ul class="footer-list">
              <li><a href="#/category/Ноутбуки" @click.prevent="selectCategory('Ноутбуки')">Ноутбуки</a></li>
              <li><a href="#/category/Смартфони" @click.prevent="selectCategory('Смартфони')">Смартфони</a></li>
              <li><a href="#/category/Планшети" @click.prevent="selectCategory('Планшети')">Планшети</a></li>
              <li><a href="#/category/Аудіо" @click.prevent="selectCategory('Аудіо')">Навушники та аудіо</a></li>
              <li><a href="#/category/Смарт-годинники" @click.prevent="selectCategory('Смарт-годинники')">Смарт-годинники</a></li>
              <li><a href="#/category/Монітори" @click.prevent="selectCategory('Монітори')">Монітори</a></li>
              <li><a href="#/category/Периферія" @click.prevent="selectCategory('Периферія')">Периферія та аксесуари</a></li>
            </ul>
          </div>

          <div class="footer-col">
            <h4 class="footer-title">Покупцям</h4>
            <ul class="footer-list">
              <li><a href="#/page/delivery" @click.prevent="openInfoPage('delivery')">Доставка і оплата</a></li>
              <li><a href="#/page/warranty" @click.prevent="openInfoPage('warranty')">Гарантія та повернення</a></li>
              <li><a href="#/page/privacy" @click.prevent="openInfoPage('privacy')">Політика конфіденційності</a></li>
              <li><a href="#/page/terms" @click.prevent="openInfoPage('terms')">Договір публічної оферти</a></li>
              <li><a href="#/page/about" @click.prevent="openInfoPage('about')">Про магазин та контакти</a></li>
            </ul>
          </div>

          <div class="footer-col">
            <h4 class="footer-title">Контакти та інформація</h4>
            <ul class="footer-list">
              <li><strong>ФОП Аверкієв І.О.</strong></li>
              <li>Тел.: <a href="tel:0670001234" class="footer-phone">0670001234</a></li>
              <li>Email: info@megatrend.com.ua</li>
              <li>Графік роботи: Пн-Нд 09:00-20:00</li>
            </ul>
          </div>
        </div>

        <div class="footer-bottom">
          <span>&copy; 2026 МегаТренд - магазин техніки.</span>
          <span>Усі права захищено</span>
        </div>
      </div>
    </footer>

    <CartDrawer
      :is-open="isCartOpen"
      :items="cart"
      :cart-count="cartCount"
      :cart-total="cartTotal"
      :discount-amount="discountAmount"
      :final-total="finalTotal"
      :applied-promo="appliedPromo"
      :promo-error="promoError"
      @close="isCartOpen = false"
      @update-quantity="updateCartQuantity"
      @remove-item="removeFromCart"
      @clear-cart="clearCart"
      @apply-promo="applyPromo"
      @remove-promo="removePromo"
      @open-checkout="openCheckout"
    />

    <OrderModal
      :is-open="isCheckoutOpen"
      :items="cart"
      :final-total="finalTotal"
      :discount-amount="discountAmount"
      @close="isCheckoutOpen = false"
      @order-submitted="handleOrderPlaced"
    />

    <Toast
      :message="toastMessage"
      :type="toastType"
      :visible="toastVisible"
    />
  </div>
</template>

<style scoped>
.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main {
  flex: 1;
  padding-top: 20px;
  padding-bottom: 44px;
}

.page-fade-enter-active,
.page-fade-leave-active {
  transition: opacity 0.24s cubic-bezier(0.22, 1, 0.36, 1), transform 0.24s cubic-bezier(0.22, 1, 0.36, 1);
}

.page-fade-enter-from,
.page-fade-leave-to {
  opacity: 0;
  transform: translateY(6px);
}

.catalog-dropdown-overlay {
  position: fixed;
  inset: 76px 0 0 0;
  background-color: rgba(10, 20, 40, 0.5);
  backdrop-filter: blur(2px);
  z-index: 65;
  padding: 14px 0 28px;
  overflow-y: auto;
}

.catalog-modal-wrap {
  max-height: 100%;
}

.catalog-mega-menu {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 20px 24px 24px;
  box-shadow: 0 18px 42px rgba(10, 20, 40, 0.22);
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.mega-dropdown-enter-active,
.mega-dropdown-leave-active {
  transition: opacity 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.mega-dropdown-enter-active .catalog-mega-menu,
.mega-dropdown-leave-active .catalog-mega-menu {
  transition: transform 0.3s cubic-bezier(0.22, 1, 0.36, 1), opacity 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.mega-dropdown-enter-from,
.mega-dropdown-leave-to {
  opacity: 0;
}

.mega-dropdown-enter-from .catalog-mega-menu,
.mega-dropdown-leave-to .catalog-mega-menu {
  transform: translateY(-10px) scale(0.985);
  opacity: 0;
}

.mega-top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 14px;
  border-bottom: 1px solid var(--border);
}

.mega-top-title {
  font-size: 18px;
  font-weight: 800;
  color: var(--navy);
}

.mega-body-grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 28px;
}

.mega-heading {
  font-size: 15px;
  font-weight: 700;
  color: var(--navy);
  margin-bottom: 12px;
}

.mega-cats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(210px, 1fr));
  gap: 10px;
}

.mega-cat-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 12px;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  text-align: left;
  background-color: var(--bg);
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}

.mega-cat-icon-box {
  width: 38px;
  height: 38px;
  border-radius: 9px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  color: var(--blue);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}

.mega-cat-item:hover,
.mega-cat-item.active {
  border-color: var(--blue);
  background-color: var(--blue-light);
  transform: translateY(-1px);
}

.mega-cat-item:hover .mega-cat-icon-box,
.mega-cat-item.active .mega-cat-icon-box {
  background-color: var(--navy);
  border-color: var(--navy);
  color: #ffffff;
}

.mega-cat-text {
  display: flex;
  flex-direction: column;
}

.mega-cat-text strong {
  font-size: 14px;
  color: var(--navy);
}

.mega-cat-text span {
  font-size: 12px;
  color: var(--text-secondary);
}

.mega-brands-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.mega-brand-chip {
  padding: 7px 12px;
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: 6px;
  font-size: 13px;
  font-weight: 600;
  color: var(--navy);
  transition: all 0.24s cubic-bezier(0.22, 1, 0.36, 1);
}

.mega-brand-chip span {
  color: var(--text-muted);
  font-weight: 500;
}

.mega-brand-chip:hover,
.mega-brand-chip.active {
  background-color: var(--navy);
  border-color: var(--navy);
  color: #ffffff;
  transform: translateY(-1px);
}

.mega-brand-chip.active span,
.mega-brand-chip:hover span {
  color: #cbd5e1;
}

.poster-block {
  margin-bottom: 18px;
  border-radius: var(--radius-lg);
  overflow: hidden;
  box-shadow: var(--shadow-sm);
  cursor: pointer;
  border: 1px solid var(--border);
  background-color: var(--navy);
  transition: transform 0.32s cubic-bezier(0.22, 1, 0.36, 1), box-shadow 0.32s cubic-bezier(0.22, 1, 0.36, 1);
}

.poster-block:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

.poster-img {
  width: 100%;
  height: auto;
  display: block;
}

.rozetka-categories {
  display: grid;
  grid-template-columns: repeat(9, 1fr);
  gap: 10px;
  margin-bottom: 16px;
}

.rz-cat-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  gap: 9px;
  padding: 13px 8px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  box-shadow: var(--shadow-sm);
  transition: all 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.rz-cat-icon-box {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background-color: var(--blue-light);
  color: var(--blue);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.rz-cat-card:hover {
  border-color: var(--blue);
  transform: translateY(-3px);
  box-shadow: var(--shadow-md);
}

.rz-cat-card:hover .rz-cat-icon-box {
  background-color: var(--blue);
  color: #ffffff;
  transform: scale(1.06);
}

.rz-cat-card.active {
  background-color: var(--navy);
  border-color: var(--navy);
  color: #ffffff;
  box-shadow: var(--shadow-md);
}

.rz-cat-card.active .rz-cat-icon-box {
  background-color: rgba(255, 255, 255, 0.14);
  color: #49e4f4;
}

.rz-cat-info {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.rz-cat-title {
  font-size: 13px;
  font-weight: 700;
  color: var(--navy);
  transition: color 0.24s ease;
}

.rz-cat-card.active .rz-cat-title {
  color: #ffffff;
}

.rz-cat-count {
  font-size: 11px;
  color: var(--text-muted);
  transition: color 0.24s ease;
}

.rz-cat-card.active .rz-cat-count {
  color: #b8cae0;
}

.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 12px 16px;
  margin-bottom: 16px;
  box-shadow: var(--shadow-sm);
}

.active-filters {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
}

.mobile-filters-btn {
  display: none;
  align-items: center;
  gap: 6px;
  height: 34px;
  padding: 0 12px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
}

.mobile-filters-badge {
  background-color: var(--orange);
  color: #ffffff;
  font-size: 11px;
  font-weight: 700;
  min-width: 18px;
  height: 18px;
  padding: 0 5px;
  border-radius: 99px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.found-count {
  font-size: 13px;
  color: var(--text-secondary);
  margin-right: 4px;
}

.filter-chip {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background-color: var(--blue-light);
  color: var(--navy);
  font-size: 12px;
  padding: 4px 10px;
  border-radius: 99px;
}

.filter-chip button {
  font-size: 11px;
  color: var(--text-secondary);
  font-weight: 700;
}

.filter-chip button:hover {
  color: var(--red);
}

.clear-all-chips-btn {
  font-size: 12px;
  font-weight: 700;
  color: var(--red);
  text-decoration: underline;
  padding: 2px 6px;
}

.toolbar-right {
  display: flex;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
}

.checkbox-label {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  font-size: 13px;
  font-weight: 500;
  color: var(--text-secondary);
  cursor: pointer;
  user-select: none;
}

.checkbox-label input {
  width: 15px;
  height: 15px;
  accent-color: var(--blue);
  cursor: pointer;
}

.sort-select {
  height: 36px;
  padding: 0 12px;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
}

.catalog-layout {
  display: flex;
  align-items: flex-start;
  gap: 20px;
}

.catalog-results {
  flex: 1;
  min-width: 0;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 16px;
}

.loading-state,
.error-state {
  padding: 44px;
  text-align: center;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  font-size: 14px;
}

.retry-btn {
  margin-top: 12px;
  padding: 8px 16px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
}

.footer {
  border-top: 1px solid var(--border);
  background-color: var(--surface);
  padding: 36px 0 20px;
  margin-top: auto;
}

.footer-top {
  display: grid;
  grid-template-columns: 1.4fr 1fr 1fr 1fr;
  gap: 28px;
  padding-bottom: 24px;
  border-bottom: 1px solid var(--border);
}

.footer-logo {
  height: 40px;
  width: auto;
  margin-bottom: 12px;
  cursor: pointer;
}

.footer-desc {
  font-size: 13px;
  color: var(--text-secondary);
  line-height: 1.55;
  max-width: 320px;
}

.footer-title {
  font-size: 14px;
  font-weight: 700;
  color: var(--navy);
  margin-bottom: 10px;
}

.footer-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 7px;
  font-size: 13px;
  color: var(--text-secondary);
}

.footer-list a:hover {
  color: var(--blue);
}

.footer-phone {
  font-weight: 700;
  color: var(--navy);
}

.footer-bottom {
  padding-top: 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  font-size: 13px;
  color: var(--text-muted);
}

@media (max-width: 1080px) {
  .rozetka-categories {
    grid-template-columns: repeat(auto-fill, minmax(125px, 1fr));
  }

  .mega-body-grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }

  .footer-top {
    grid-template-columns: 1fr 1fr;
  }
}

@media (max-width: 920px) {
  .mobile-filters-btn {
    display: inline-flex;
  }

  .catalog-layout {
    gap: 0;
  }
}

@media (max-width: 780px) {
  .catalog-dropdown-overlay {
    inset: 112px 0 0 0;
    padding: 10px 0 20px;
  }

  .catalog-mega-menu {
    padding: 16px;
    border-radius: var(--radius);
  }

  .mega-cats-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .rozetka-categories {
    display: flex;
    overflow-x: auto;
    padding-bottom: 6px;
    gap: 8px;
  }

  .rz-cat-card {
    min-width: 118px;
    flex-shrink: 0;
  }

  .toolbar {
    flex-direction: column;
    align-items: stretch;
  }

  .toolbar-right {
    justify-content: space-between;
  }

  .grid {
    grid-template-columns: 1fr;
  }

  .footer-top {
    grid-template-columns: 1fr;
    gap: 20px;
  }
}
</style>

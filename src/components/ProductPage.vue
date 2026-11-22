<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import ProductCard from './ProductCard.vue'

const props = defineProps({
  product: {
    type: Object,
    required: true
  },
  allProducts: {
    type: Array,
    default: () => []
  },
  autoScrollReviews: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits([
  'back',
  'addToCart',
  'openProduct',
  'selectCategory',
  'selectBrand',
  'addReview'
])

const added = ref(false)
const activeImageIndex = ref(0)
const isLightboxOpen = ref(false)
const reviewsSectionRef = ref(null)

const isReviewFormOpen = ref(false)
const reviewAuthor = ref('')
const reviewRating = ref(5)
const hoverRating = ref(0)
const reviewText = ref('')
const reviewError = ref('')

const ratingLabels = {
  1: '1 з 5 - Дуже погано',
  2: '2 з 5 - Погано',
  3: '3 з 5 - Нормально',
  4: '4 з 5 - Добре',
  5: '5 з 5 - Відмінно'
}

const galleryImages = computed(() => {
  if (props.product.images && props.product.images.length > 0) {
    return props.product.images
  }
  return [props.product.image]
})

const activeImage = computed(() => {
  return galleryImages.value[activeImageIndex.value] || props.product.image
})

watch(
  () => props.product,
  () => {
    activeImageIndex.value = 0
    isLightboxOpen.value = false
    isReviewFormOpen.value = false
    reviewError.value = ''
    if (props.autoScrollReviews) {
      setTimeout(() => {
        scrollToReviews()
      }, 100)
    }
  },
  { immediate: true }
)

watch(isLightboxOpen, (open) => {
  if (open) {
    document.body.style.overflow = 'hidden'
  } else {
    document.body.style.overflow = ''
  }
})

onMounted(() => {
  if (props.autoScrollReviews) {
    setTimeout(() => {
      scrollToReviews()
    }, 120)
  }
})

function formatCurrency(amount) {
  return new Intl.NumberFormat('uk-UA').format(amount) + ' ₴'
}

const discountPercent = computed(() => {
  if (!props.product.oldPrice || props.product.oldPrice <= props.product.price) return 0
  return Math.round(((props.product.oldPrice - props.product.price) / props.product.oldPrice) * 100)
})

const relatedProducts = computed(() => {
  return props.allProducts
    .filter((p) => p.category === props.product.category && p.id !== props.product.id)
    .slice(0, 4)
})

function selectImageIndex(idx) {
  activeImageIndex.value = idx
}

function openLightbox() {
  isLightboxOpen.value = true
}

function closeLightbox() {
  isLightboxOpen.value = false
}

function prevImage() {
  const len = galleryImages.value.length
  activeImageIndex.value = (activeImageIndex.value - 1 + len) % len
}

function nextImage() {
  const len = galleryImages.value.length
  activeImageIndex.value = (activeImageIndex.value + 1) % len
}

function handleAdd() {
  emit('addToCart', props.product)
  added.value = true
  setTimeout(() => {
    added.value = false
  }, 1200)
}

function scrollToReviews() {
  if (reviewsSectionRef.value) {
    reviewsSectionRef.value.scrollIntoView({ behavior: 'smooth', block: 'start' })
  }
}

function toggleReviewForm() {
  isReviewFormOpen.value = !isReviewFormOpen.value
  reviewError.value = ''
}

function submitReview() {
  const cleanName = reviewAuthor.value.trim()
  if (cleanName.length < 2) {
    reviewError.value = 'Будь ласка, введіть ваше ім\'я (мінімум 2 символи)'
    return
  }

  emit('addReview', {
    productId: props.product.id,
    author: cleanName,
    rating: reviewRating.value,
    text: reviewText.value.trim()
  })

  reviewAuthor.value = ''
  reviewRating.value = 5
  hoverRating.value = 0
  reviewText.value = ''
  reviewError.value = ''
  isReviewFormOpen.value = false
}

function getStarsString(rating) {
  const full = Math.round(rating)
  return '★'.repeat(full) + '☆'.repeat(5 - full)
}

function handleImageError(e) {
  e.target.src = 'https://i.moyo.ua/img/products/7376/99_600.jpg'
}
</script>

<template>
  <section class="product-page">
    <nav class="breadcrumbs">
      <button type="button" class="crumb-link" @click="emit('back')">Каталог</button>
      <span class="crumb-sep">/</span>
      <button type="button" class="crumb-link" @click="emit('selectCategory', product.category)">
        {{ product.category }}
      </button>
      <span v-if="product.brand" class="crumb-sep">/</span>
      <button
        v-if="product.brand"
        type="button"
        class="crumb-link"
        @click="emit('selectBrand', product.brand)"
      >
        {{ product.brand }}
      </button>
      <span class="crumb-sep">/</span>
      <span class="crumb-current">{{ product.name }}</span>
    </nav>

    <div class="back-row">
      <button type="button" class="back-btn" @click="emit('back')">
        ← Повернутися до списку товарів
      </button>
    </div>

    <div class="product-main">
      <div class="gallery-panel">
        <div v-if="product.badge || discountPercent > 0" class="gallery-top-bar">
          <div class="gallery-badges">
            <span v-if="product.badge" class="badge" :class="product.oldPrice ? 'badge-sale' : 'badge-hit'">
              {{ product.badge }}
            </span>
            <span v-if="discountPercent > 0" class="badge badge-discount">
              -{{ discountPercent }}%
            </span>
          </div>
        </div>

        <div class="main-img-wrap" @click="openLightbox" title="Натисніть для збільшення фото">
          <Transition name="img-crossfade" mode="out-in">
            <img
              :key="activeImage"
              :src="activeImage"
              :alt="product.name"
              class="main-img"
              @error="handleImageError"
            />
          </Transition>
        </div>

        <div v-if="galleryImages.length > 1" class="thumbs-row">
          <button
            v-for="(imgUrl, index) in galleryImages"
            :key="index"
            type="button"
            class="thumb-btn"
            :class="{ active: activeImageIndex === index }"
            @click="selectImageIndex(index)"
          >
            <img :src="imgUrl" :alt="product.name + ' ракурс ' + (index + 1)" @error="handleImageError" />
          </button>
        </div>
      </div>

      <div class="info-panel">
        <div class="info-top-meta">
          <span class="brand-tag">
            Бренд:
            <button
              type="button"
              class="brand-link-btn"
              @click="emit('selectBrand', product.brand || product.category)"
            >
              {{ product.brand || product.category }}
            </button>
          </span>
          <span class="sku-tag">Код товару: <strong>{{ product.sku }}</strong></span>
        </div>

        <h1 class="page-title">{{ product.name }}</h1>

        <div class="rating-bar">
          <button type="button" class="reviews-trigger" @click="scrollToReviews">
            <span class="star-icon">★★★★★</span>
            <span class="rating-val">{{ product.rating }}</span>
            <span class="reviews-text">({{ product.reviewsCount }} відгуків покупців)</span>
          </button>

          <span v-if="product.inStock" class="status-pill in">● В наявності</span>
          <span v-else class="status-pill out">Немає в наявності</span>
        </div>

        <div class="buy-card">
          <div class="buy-price-block">
            <span v-if="product.oldPrice" class="buy-old-price">{{ formatCurrency(product.oldPrice) }}</span>
            <span class="buy-current-price" :class="{ 'is-sale': product.oldPrice }">
              {{ formatCurrency(product.price) }}
            </span>
          </div>

          <button
            type="button"
            class="buy-main-btn"
            :class="{ 'is-added': added }"
            :disabled="!product.inStock"
            @click="handleAdd"
          >
            {{ added ? 'Додано в кошик ✓' : 'Купити' }}
          </button>
        </div>

        <div class="description-block">
          <h2 class="sub-heading">Опис товару</h2>
          <p class="full-desc">{{ product.fullDescription || product.description }}</p>
        </div>

        <div class="delivery-grid">
          <div class="delivery-item">
            <strong>🚚 Доставка по Україні</strong>
            <span>Нова Пошта у відділення, поштомат або кур'єром (1-2 дні)</span>
          </div>
          <div class="delivery-item">
            <strong>💳 Оплата</strong>
            <span>Оплата під час отримання, карткою онлайн або за реквізитами ФОП</span>
          </div>
          <div class="delivery-item">
            <strong>🛡️ Гарантія</strong>
            <span>{{ product.warranty || '12 місяців офіційної гарантії' }}</span>
          </div>
        </div>
      </div>
    </div>

    <div v-if="product.specs" class="specs-section">
      <h2 class="section-heading">Технічні характеристики {{ product.name }}</h2>
      <div class="specs-table">
        <div
          v-for="(val, key) in product.specs"
          :key="key"
          class="spec-row"
        >
          <span class="spec-key">{{ key }}</span>
          <span class="spec-val">{{ val }}</span>
        </div>
      </div>
    </div>

    <div ref="reviewsSectionRef" class="reviews-section">
      <div class="reviews-header">
        <h2 class="section-heading">Відгуки покупців ({{ product.reviewsCount }})</h2>
        <div class="reviews-header-actions">
          <div class="reviews-summary-badge">
            <span class="summary-stars">★★★★★</span>
            <strong>{{ product.rating }} з 5</strong>
          </div>
          <button
            type="button"
            class="write-review-btn"
            :class="{ active: isReviewFormOpen }"
            @click="toggleReviewForm"
          >
            {{ isReviewFormOpen ? '✕ Закрити форму' : '+ Написати відгук' }}
          </button>
        </div>
      </div>

      <Transition name="review-form-slide">
        <form v-if="isReviewFormOpen" class="review-form-card" @submit.prevent="submitReview">
          <h3 class="review-form-title">Залишити відгук про {{ product.name }}</h3>

          <div class="review-form-row">
            <div class="review-field">
              <label class="review-label" for="rev-author">Ваше ім'я або ім'я та прізвище *</label>
              <input
                id="rev-author"
                v-model="reviewAuthor"
                type="text"
                class="review-input"
                placeholder="Наприклад: Іван Петренко"
                maxlength="60"
                @input="reviewError = ''"
              />
            </div>

            <div class="review-field">
              <label class="review-label">Ваша оцінка *</label>
              <div class="star-picker-wrap">
                <div class="star-picker" @mouseleave="hoverRating = 0">
                  <button
                    v-for="star in 5"
                    :key="star"
                    type="button"
                    class="star-pick-btn"
                    :class="{ filled: star <= (hoverRating || reviewRating) }"
                    :aria-label="'Оцінка ' + star + ' з 5'"
                    @mouseenter="hoverRating = star"
                    @click="reviewRating = star"
                  >
                    ★
                  </button>
                </div>
                <span class="star-picker-label">
                  {{ ratingLabels[hoverRating || reviewRating] }}
                </span>
              </div>
            </div>
          </div>

          <div class="review-field">
            <label class="review-label" for="rev-text">Коментар про товар (необов'язково)</label>
            <textarea
              id="rev-text"
              v-model="reviewText"
              class="review-textarea"
              rows="3"
              maxlength="500"
              placeholder="Поділіться враженнями від користування товаром..."
            ></textarea>
          </div>

          <p v-if="reviewError" class="review-error-msg">{{ reviewError }}</p>

          <div class="review-form-actions">
            <button type="button" class="review-cancel-btn" @click="toggleReviewForm">
              Скасувати
            </button>
            <button type="submit" class="review-submit-btn">
              Опублікувати відгук
            </button>
          </div>
        </form>
      </Transition>

      <TransitionGroup
        v-if="product.reviews && product.reviews.length > 0"
        name="review-list"
        tag="div"
        class="reviews-grid"
      >
        <div
          v-for="rev in product.reviews"
          :key="rev.id"
          class="review-card"
        >
          <div class="review-card-top">
            <div class="review-user">
              <div class="user-avatar">{{ rev.author.charAt(0).toUpperCase() }}</div>
              <div class="user-meta">
                <strong class="user-name">{{ rev.author }}</strong>
                <span class="review-date">{{ rev.date }}</span>
              </div>
            </div>
            <div class="review-stars" :title="rev.rating + ' з 5'">
              {{ getStarsString(rev.rating) }}
            </div>
          </div>
          <p v-if="rev.text" class="review-comment">{{ rev.text }}</p>
        </div>
      </TransitionGroup>
    </div>

    <div v-if="relatedProducts.length > 0" class="related-section">
      <h2 class="section-heading">Інші товари з категорії «{{ product.category }}»</h2>
      <div class="related-grid">
        <ProductCard
          v-for="item in relatedProducts"
          :key="item.id"
          :product="item"
          @add-to-cart="emit('addToCart', $event)"
          @open-product="emit('openProduct', $event)"
          @open-reviews="emit('openProduct', $event)"
          @select-brand="emit('selectBrand', $event)"
        />
      </div>
    </div>

    <Transition name="lightbox">
      <div v-if="isLightboxOpen" class="lightbox-overlay" @click="closeLightbox" @wheel.prevent>
        <div class="lightbox-modal" @click.stop>
          <div class="lightbox-header">
            <div class="lightbox-title-wrap">
              <h3 class="lightbox-title">{{ product.name }}</h3>
              <span class="lightbox-counter">Фото {{ activeImageIndex + 1 }} з {{ galleryImages.length }}</span>
            </div>
            <button type="button" class="lightbox-close" @click="closeLightbox" aria-label="Закрити">
              ✕
            </button>
          </div>

          <div class="lightbox-stage">
            <button
              v-if="galleryImages.length > 1"
              type="button"
              class="lightbox-nav prev"
              @click="prevImage"
              aria-label="Попереднє фото"
            >
              ‹
            </button>

            <div class="lightbox-img-container">
              <Transition name="img-crossfade" mode="out-in">
                <img
                  :key="activeImage"
                  :src="activeImage"
                  :alt="product.name"
                  class="lightbox-img"
                  @error="handleImageError"
                />
              </Transition>
            </div>

            <button
              v-if="galleryImages.length > 1"
              type="button"
              class="lightbox-nav next"
              @click="nextImage"
              aria-label="Наступне фото"
            >
              ›
            </button>
          </div>

          <div class="lightbox-footer">
            <div v-if="galleryImages.length > 1" class="lightbox-thumbs">
              <button
                v-for="(imgUrl, index) in galleryImages"
                :key="index"
                type="button"
                class="lightbox-thumb-btn"
                :class="{ active: activeImageIndex === index }"
                @click="selectImageIndex(index)"
              >
                <img :src="imgUrl" :alt="'Мініатюра ' + (index + 1)" @error="handleImageError" />
              </button>
            </div>

            <div class="lightbox-buy-bar">
              <span class="lightbox-price">{{ formatCurrency(product.price) }}</span>
              <button
                type="button"
                class="buy-main-btn"
                :class="{ 'is-added': added }"
                @click="handleAdd"
              >
                {{ added ? 'У кошику ✓' : 'Купити' }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </section>
</template>

<style scoped>
.product-page {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.breadcrumbs {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
  font-size: 13px;
  color: var(--text-muted);
}

.crumb-link {
  color: var(--text-secondary);
  font-weight: 500;
}

.crumb-link:hover {
  color: var(--blue);
  text-decoration: underline;
}

.crumb-sep {
  color: var(--border-hover);
}

.crumb-current {
  color: var(--text);
  font-weight: 500;
}

.back-row {
  display: flex;
}

.back-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  height: 36px;
  padding: 0 14px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
  color: var(--navy);
}

.back-btn:hover {
  border-color: var(--blue);
  color: var(--blue);
  transform: translateX(-2px);
}

.product-main {
  display: grid;
  grid-template-columns: 440px 1fr;
  gap: 24px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 24px;
  box-shadow: var(--shadow-sm);
}

.gallery-panel {
  display: flex;
  flex-direction: column;
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 16px;
  background-color: #ffffff;
}

.gallery-top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  margin-bottom: 12px;
}

.gallery-badges {
  display: flex;
  gap: 8px;
}

.badge {
  font-size: 12px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 4px;
}

.badge-hit {
  background-color: var(--blue-light);
  color: var(--blue);
}

.badge-sale {
  background-color: var(--orange-light);
  color: var(--orange);
}

.badge-discount {
  background-color: var(--red-bg);
  color: var(--red);
}

.main-img-wrap {
  height: 320px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 12px;
  cursor: zoom-in;
}

.main-img {
  max-height: 100%;
  max-width: 100%;
  object-fit: contain;
  transition: transform 0.34s cubic-bezier(0.22, 1, 0.36, 1);
}

.main-img-wrap:hover .main-img {
  transform: scale(1.03);
}

.img-crossfade-enter-active,
.img-crossfade-leave-active {
  transition: opacity 0.22s cubic-bezier(0.22, 1, 0.36, 1), transform 0.22s cubic-bezier(0.22, 1, 0.36, 1);
}

.img-crossfade-enter-from,
.img-crossfade-leave-to {
  opacity: 0;
  transform: scale(0.97);
}

.thumbs-row {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  margin-top: 14px;
  padding-top: 14px;
  border-top: 1px solid var(--border);
  flex-wrap: wrap;
}

.thumb-btn {
  width: 64px;
  height: 64px;
  padding: 6px;
  border: 2px solid var(--border);
  border-radius: var(--radius-sm);
  background-color: #ffffff;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.24s cubic-bezier(0.22, 1, 0.36, 1);
}

.thumb-btn:hover {
  border-color: var(--border-hover);
  transform: translateY(-2px);
}

.thumb-btn.active {
  border-color: var(--blue);
  box-shadow: 0 0 0 3px rgba(21, 158, 216, 0.14);
}

.thumb-btn img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.info-panel {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.info-top-meta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
  font-size: 13px;
  color: var(--text-secondary);
}

.brand-link-btn {
  color: var(--blue);
  font-weight: 700;
  text-decoration: underline;
  padding: 2px 4px;
}

.brand-link-btn:hover {
  color: var(--navy);
}

.page-title {
  font-size: 24px;
  font-weight: 800;
  color: var(--navy);
  line-height: 1.3;
}

.rating-bar {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 14px;
  padding-bottom: 14px;
  border-bottom: 1px solid var(--border);
  font-size: 13px;
}

.reviews-trigger {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 4px 8px;
  margin-left: -8px;
  border-radius: 6px;
}

.reviews-trigger:hover {
  background-color: var(--bg);
}

.star-icon {
  color: #f59e0b;
  letter-spacing: 1px;
}

.rating-val {
  font-weight: 700;
  color: var(--navy);
}

.reviews-text {
  color: var(--blue);
  text-decoration: underline;
  font-weight: 500;
}

.status-pill {
  margin-left: auto;
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 12px;
  font-weight: 600;
}

.status-pill.in {
  background-color: var(--green-bg);
  color: var(--green);
}

.status-pill.out {
  background-color: var(--bg);
  color: var(--text-muted);
}

.buy-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 16px;
  padding: 18px 20px;
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: var(--radius);
}

.buy-price-block {
  display: flex;
  flex-direction: column;
}

.buy-old-price {
  font-size: 14px;
  color: var(--text-muted);
  text-decoration: line-through;
}

.buy-current-price {
  font-size: 28px;
  font-weight: 800;
  color: var(--navy);
  line-height: 1.15;
}

.buy-current-price.is-sale {
  color: var(--red);
}

.buy-main-btn {
  height: 46px;
  padding: 0 28px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 15px;
  font-weight: 700;
}

.buy-main-btn:hover:not(:disabled) {
  background-color: var(--blue);
  transform: translateY(-1px);
}

.buy-main-btn.is-added {
  background-color: var(--green);
}

.sub-heading {
  font-size: 16px;
  font-weight: 700;
  color: var(--navy);
  margin-bottom: 6px;
}

.full-desc {
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.6;
}

.delivery-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin-top: 4px;
}

.delivery-item {
  padding: 12px;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.delivery-item strong {
  font-size: 13px;
  color: var(--navy);
}

.delivery-item span {
  font-size: 12px;
  color: var(--text-secondary);
  line-height: 1.4;
}

.specs-section,
.reviews-section {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 24px;
  box-shadow: var(--shadow-sm);
}

.section-heading {
  font-size: 18px;
  font-weight: 700;
  color: var(--navy);
  margin-bottom: 16px;
}

.specs-table {
  display: flex;
  flex-direction: column;
}

.spec-row {
  display: grid;
  grid-template-columns: 300px 1fr;
  gap: 16px;
  padding: 11px 14px;
  font-size: 14px;
  border-radius: 6px;
}

.spec-row:nth-child(odd) {
  background-color: var(--bg);
}

.spec-key {
  color: var(--text-secondary);
  font-weight: 500;
}

.spec-val {
  color: var(--navy);
  font-weight: 600;
}

.reviews-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 18px;
}

.reviews-header .section-heading {
  margin-bottom: 0;
}

.reviews-header-actions {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}

.reviews-summary-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background-color: var(--bg);
  border: 1px solid var(--border);
  padding: 6px 14px;
  border-radius: 99px;
  font-size: 14px;
  color: var(--navy);
}

.summary-stars {
  color: #f59e0b;
  letter-spacing: 1px;
}

.write-review-btn {
  height: 36px;
  padding: 0 16px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: 99px;
  font-size: 13px;
  font-weight: 700;
  transition: all 0.25s cubic-bezier(0.22, 1, 0.36, 1);
}

.write-review-btn:hover {
  background-color: var(--blue);
  transform: translateY(-1px);
}

.write-review-btn.active {
  background-color: var(--bg);
  color: var(--navy);
  border: 1px solid var(--border);
}

.review-form-card {
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 18px 20px;
  margin-bottom: 18px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.review-form-slide-enter-active,
.review-form-slide-leave-active {
  transition: all 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.review-form-slide-enter-from,
.review-form-slide-leave-to {
  opacity: 0;
  transform: translateY(-8px);
}

.review-form-title {
  font-size: 15px;
  font-weight: 700;
  color: var(--navy);
}

.review-form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.review-field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.review-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
}

.review-input {
  height: 40px;
  padding: 0 12px;
  font-size: 14px;
  background-color: #ffffff;
}

.review-textarea {
  padding: 10px 12px;
  font-size: 14px;
  background-color: #ffffff;
  resize: vertical;
  min-height: 76px;
}

.star-picker-wrap {
  display: flex;
  align-items: center;
  gap: 12px;
  height: 40px;
  padding: 0 12px;
  background-color: #ffffff;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
}

.star-picker {
  display: inline-flex;
  align-items: center;
  gap: 2px;
}

.star-pick-btn {
  font-size: 22px;
  line-height: 1;
  color: #cbd5e1;
  padding: 2px;
  transition: transform 0.18s cubic-bezier(0.22, 1, 0.36, 1), color 0.18s ease;
}

.star-pick-btn.filled {
  color: #f59e0b;
}

.star-pick-btn:hover {
  transform: scale(1.18);
}

.star-picker-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--navy);
}

.review-error-msg {
  font-size: 13px;
  font-weight: 600;
  color: var(--red);
}

.review-form-actions {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 10px;
}

.review-cancel-btn {
  height: 38px;
  padding: 0 16px;
  background-color: #ffffff;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
  color: var(--text-secondary);
}

.review-cancel-btn:hover {
  border-color: var(--navy);
  color: var(--navy);
}

.review-submit-btn {
  height: 38px;
  padding: 0 20px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 700;
}

.review-submit-btn:hover {
  background-color: var(--blue);
  transform: translateY(-1px);
}

.reviews-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 14px;
}

.review-list-enter-active,
.review-list-leave-active {
  transition: all 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.review-list-enter-from,
.review-list-leave-to {
  opacity: 0;
  transform: translateY(10px) scale(0.97);
}

.review-card {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 10px;
  padding: 14px 16px;
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: var(--radius);
}

.review-card-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.review-user {
  display: flex;
  align-items: center;
  gap: 10px;
}

.user-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background-color: var(--navy);
  color: #ffffff;
  font-size: 14px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.user-meta {
  display: flex;
  flex-direction: column;
}

.user-name {
  font-size: 14px;
  font-weight: 600;
  color: var(--navy);
}

.review-date {
  font-size: 12px;
  color: var(--text-muted);
}

.review-stars {
  color: #f59e0b;
  font-size: 16px;
  letter-spacing: 1px;
  flex-shrink: 0;
}

.review-comment {
  font-size: 13px;
  color: var(--text-secondary);
  line-height: 1.5;
  padding-top: 8px;
  border-top: 1px solid var(--border);
  word-break: break-word;
}

.related-section {
  margin-top: 8px;
}

.related-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(260px, 1fr));
  gap: 16px;
}

.lightbox-overlay {
  position: fixed;
  inset: 0;
  z-index: 80;
  background-color: rgba(10, 20, 40, 0.72);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
}

.lightbox-modal {
  width: 100%;
  max-width: 960px;
  max-height: 92vh;
  background-color: #ffffff;
  border-radius: var(--radius-lg);
  box-shadow: 0 24px 54px rgba(0, 0, 0, 0.35);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.lightbox-enter-active,
.lightbox-leave-active {
  transition: opacity 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.lightbox-enter-active .lightbox-modal,
.lightbox-leave-active .lightbox-modal {
  transition: transform 0.32s cubic-bezier(0.22, 1, 0.36, 1), opacity 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.lightbox-enter-from,
.lightbox-leave-to {
  opacity: 0;
}

.lightbox-enter-from .lightbox-modal,
.lightbox-leave-to .lightbox-modal {
  transform: scale(0.94) translateY(10px);
  opacity: 0;
}

.lightbox-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 16px 22px;
  border-bottom: 1px solid var(--border);
}

.lightbox-title-wrap {
  display: flex;
  align-items: baseline;
  flex-wrap: wrap;
  gap: 12px;
}

.lightbox-title {
  font-size: 17px;
  font-weight: 700;
  color: var(--navy);
}

.lightbox-counter {
  font-size: 13px;
  color: var(--text-muted);
  font-weight: 500;
}

.lightbox-close {
  width: 36px;
  height: 36px;
  border-radius: 8px;
  background-color: var(--bg);
  color: var(--navy);
  font-size: 16px;
  font-weight: 700;
  flex-shrink: 0;
}

.lightbox-close:hover {
  background-color: var(--red-bg);
  color: var(--red);
}

.lightbox-stage {
  position: relative;
  flex: 1;
  min-height: 360px;
  max-height: 64vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px 64px;
  background-color: #ffffff;
}

.lightbox-img-container {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lightbox-img {
  max-width: 100%;
  max-height: 58vh;
  object-fit: contain;
}

.lightbox-nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background-color: var(--bg);
  border: 1px solid var(--border);
  color: var(--navy);
  font-size: 28px;
  line-height: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: var(--shadow-sm);
}

.lightbox-nav:hover {
  background-color: var(--navy);
  color: #ffffff;
  border-color: var(--navy);
  transform: translateY(-50%) scale(1.06);
}

.lightbox-nav.prev {
  left: 16px;
}

.lightbox-nav.next {
  right: 16px;
}

.lightbox-footer {
  padding: 14px 22px;
  border-top: 1px solid var(--border);
  background-color: var(--bg);
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 16px;
}

.lightbox-thumbs {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.lightbox-thumb-btn {
  width: 58px;
  height: 58px;
  padding: 5px;
  background-color: #ffffff;
  border: 2px solid var(--border);
  border-radius: var(--radius-sm);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.24s cubic-bezier(0.22, 1, 0.36, 1);
}

.lightbox-thumb-btn:hover {
  transform: translateY(-2px);
}

.lightbox-thumb-btn.active {
  border-color: var(--blue);
  box-shadow: 0 0 0 3px rgba(21, 158, 216, 0.14);
}

.lightbox-thumb-btn img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.lightbox-buy-bar {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-left: auto;
}

.lightbox-price {
  font-size: 22px;
  font-weight: 800;
  color: var(--navy);
}

@media (max-width: 920px) {
  .product-main {
    grid-template-columns: 1fr;
    padding: 18px;
  }

  .main-img-wrap {
    height: 250px;
  }

  .delivery-grid {
    grid-template-columns: 1fr;
  }

  .spec-row {
    grid-template-columns: 1fr;
    gap: 4px;
  }

  .review-form-row {
    grid-template-columns: 1fr;
  }

  .lightbox-stage {
    padding: 16px 48px;
    min-height: 260px;
  }

  .lightbox-nav {
    width: 36px;
    height: 36px;
    font-size: 22px;
  }

  .lightbox-nav.prev {
    left: 8px;
  }

  .lightbox-nav.next {
    right: 8px;
  }
}
</style>

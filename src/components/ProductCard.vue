<script setup>
import { ref } from 'vue'

const props = defineProps({
  product: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['addToCart', 'openProduct', 'openReviews', 'selectBrand'])

const added = ref(false)

function formatCurrency(amount) {
  return new Intl.NumberFormat('uk-UA').format(amount) + ' ₴'
}

function handleAdd() {
  emit('addToCart', props.product)
  added.value = true
  setTimeout(() => {
    added.value = false
  }, 1100)
}

function handleOpen() {
  emit('openProduct', props.product)
}

function handleOpenReviews() {
  emit('openReviews', props.product)
}

function handleBrandClick() {
  if (props.product.brand) {
    emit('selectBrand', props.product.brand)
  }
}

function handleImageError(e) {
  e.target.src = 'https://i.moyo.ua/img/products/7376/99_600.jpg'
}
</script>

<template>
  <article class="card">
    <div class="card-top" @click="handleOpen">
      <div class="card-badges">
        <span v-if="product.badge" class="badge" :class="product.oldPrice ? 'badge-sale' : 'badge-hit'">
          {{ product.badge }}
        </span>
        <span class="sku-label">Код: {{ product.sku }}</span>
      </div>

      <div class="img-box">
        <img
          :src="product.image"
          :alt="product.name"
          class="product-img"
          loading="lazy"
          @error="handleImageError"
        />
      </div>
    </div>

    <div class="card-body">
      <div class="meta-row">
        <button
          v-if="product.brand"
          type="button"
          class="brand-btn"
          @click.stop="handleBrandClick"
        >
          {{ product.brand }}
        </button>
        <span v-else class="category-name">{{ product.category }}</span>

        <button
          type="button"
          class="rating-btn"
          @click.stop="handleOpenReviews"
          title="Переглянути відгуки"
        >
          <span class="star">★</span>
          <span class="rating-num">{{ product.rating }}</span>
          <span class="reviews">({{ product.reviewsCount }})</span>
        </button>
      </div>

      <h3 class="product-title" @click="handleOpen">
        {{ product.name }}
      </h3>

      <div class="stock-row">
        <span v-if="product.inStock" class="in-stock">● В наявності</span>
        <span v-else class="out-stock">Немає в наявності</span>
      </div>

      <div class="card-footer">
        <div class="price-col">
          <span v-if="product.oldPrice" class="old-price">{{ formatCurrency(product.oldPrice) }}</span>
          <span class="current-price" :class="{ 'has-discount': product.oldPrice }">
            {{ formatCurrency(product.price) }}
          </span>
        </div>

        <button
          type="button"
          class="buy-btn"
          :class="{ 'is-added': added }"
          :disabled="!product.inStock"
          @click.stop="handleAdd"
        >
          {{ added ? 'У кошику ✓' : 'Купити' }}
        </button>
      </div>
    </div>
  </article>
</template>

<style scoped>
.card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  display: flex;
  flex-direction: column;
  overflow: hidden;
  transition: transform 0.3s cubic-bezier(0.22, 1, 0.36, 1), border-color 0.26s ease, box-shadow 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.card:hover {
  border-color: var(--border-hover);
  box-shadow: var(--shadow-md);
  transform: translateY(-3px);
}

.card-top {
  padding: 14px 16px 8px;
  cursor: pointer;
  background-color: #ffffff;
  border-bottom: 1px solid #f0f3f8;
}

.card-badges {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
  min-height: 22px;
}

.badge {
  font-size: 11px;
  font-weight: 700;
  padding: 2px 8px;
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

.sku-label {
  font-size: 11px;
  color: var(--text-muted);
  margin-left: auto;
}

.img-box {
  height: 190px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 8px;
}

.product-img {
  max-height: 100%;
  max-width: 100%;
  width: auto;
  height: auto;
  object-fit: contain;
  transition: transform 0.36s cubic-bezier(0.22, 1, 0.36, 1);
}

.card-top:hover .product-img {
  transform: scale(1.04);
}

.card-body {
  padding: 14px 16px 16px;
  display: flex;
  flex-direction: column;
  flex: 1;
}

.meta-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 12px;
  margin-bottom: 6px;
}

.brand-btn {
  color: var(--blue);
  font-weight: 600;
  font-size: 12px;
}

.brand-btn:hover {
  text-decoration: underline;
}

.category-name {
  color: var(--text-muted);
  font-weight: 500;
}

.rating-btn {
  display: inline-flex;
  align-items: center;
  gap: 3px;
  padding: 2px 6px;
  border-radius: 4px;
}

.rating-btn:hover {
  background-color: var(--bg);
}

.star {
  color: #f59e0b;
  font-size: 13px;
}

.rating-num {
  font-weight: 600;
  color: var(--text);
}

.reviews {
  color: var(--blue);
  text-decoration: underline;
}

.product-title {
  font-size: 15px;
  font-weight: 600;
  color: var(--navy);
  line-height: 1.38;
  margin-bottom: 10px;
  cursor: pointer;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  min-height: 41px;
}

.product-title:hover {
  color: var(--blue);
}

.stock-row {
  font-size: 12px;
  margin-bottom: 12px;
}

.in-stock {
  color: var(--green);
  font-weight: 600;
}

.out-stock {
  color: var(--text-muted);
}

.card-footer {
  margin-top: auto;
  padding-top: 12px;
  border-top: 1px solid var(--border);
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 8px;
}

.price-col {
  display: flex;
  flex-direction: column;
}

.old-price {
  font-size: 12px;
  color: var(--text-muted);
  text-decoration: line-through;
  line-height: 1.1;
}

.current-price {
  font-size: 18px;
  font-weight: 800;
  color: var(--navy);
  line-height: 1.2;
}

.current-price.has-discount {
  color: var(--red);
}

.buy-btn {
  height: 36px;
  padding: 0 18px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
  white-space: nowrap;
}

.buy-btn:hover:not(:disabled) {
  background-color: var(--blue);
}

.buy-btn.is-added {
  background-color: var(--green);
}

.buy-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>

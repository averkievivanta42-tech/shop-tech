<script setup>
import logoUrl from '../megatrend_logo.svg'

defineProps({
  cartCount: {
    type: Number,
    default: 0
  },
  cartTotal: {
    type: Number,
    default: 0
  },
  searchQuery: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['toggleCart', 'update:searchQuery', 'goHome', 'toggleCatalogMenu'])

function formatCurrency(amount) {
  return new Intl.NumberFormat('uk-UA').format(amount) + ' ₴'
}
</script>

<template>
  <header class="header">
    <div class="container header-inner">
      <a href="#" class="logo-link" @click.prevent="emit('goHome')">
        <img :src="logoUrl" alt="МегаТренд - магазин техніки" class="logo-img" />
      </a>

      <button type="button" class="catalog-btn" @click="emit('toggleCatalogMenu')">
        <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2">
          <rect x="3" y="3" width="7" height="7" rx="1" />
          <rect x="14" y="3" width="7" height="7" rx="1" />
          <rect x="14" y="14" width="7" height="7" rx="1" />
          <rect x="3" y="14" width="7" height="7" rx="1" />
        </svg>
        <span>Каталог</span>
      </button>

      <div class="search-wrap">
        <svg class="search-icon" width="17" height="17" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <circle cx="11" cy="11" r="8" />
          <line x1="21" y1="21" x2="16.65" y2="16.65" />
        </svg>
        <input
          type="text"
          :value="searchQuery"
          @input="emit('update:searchQuery', $event.target.value)"
          placeholder="Я шукаю..."
          class="search-input"
        />
        <button
          v-if="searchQuery"
          type="button"
          class="clear-btn"
          @click="emit('update:searchQuery', '')"
          title="Очистити пошук"
        >
          ✕
        </button>
      </div>

      <button type="button" class="cart-btn" @click="emit('toggleCart')">
        <div class="cart-icon-box">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="9" cy="21" r="1" />
            <circle cx="20" cy="21" r="1" />
            <path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6" />
          </svg>
          <span class="cart-badge" :class="{ 'has-items': cartCount > 0 }">{{ cartCount }}</span>
        </div>
        <div class="cart-info">
          <span class="cart-title">Кошик</span>
          <span class="cart-amount">{{ cartTotal > 0 ? formatCurrency(cartTotal) : 'Порожній' }}</span>
        </div>
      </button>
    </div>
  </header>
</template>

<style scoped>
.header {
  background-color: var(--surface);
  border-bottom: 1px solid var(--border);
  position: sticky;
  top: 0;
  z-index: 40;
  box-shadow: var(--shadow-sm);
}

.header-inner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  height: 76px;
}

.logo-link {
  display: flex;
  align-items: center;
  flex-shrink: 0;
}

.logo-img {
  height: 46px;
  width: auto;
  object-fit: contain;
}

.catalog-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  height: 42px;
  padding: 0 18px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 14px;
  font-weight: 700;
  flex-shrink: 0;
}

.catalog-btn:hover {
  background-color: var(--blue);
}

.search-wrap {
  position: relative;
  flex: 1;
  max-width: 520px;
}

.search-icon {
  position: absolute;
  left: 13px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted);
  pointer-events: none;
}

.search-input {
  width: 100%;
  height: 42px;
  padding: 0 36px 0 40px;
  background-color: var(--bg);
  border-color: var(--border);
  border-radius: var(--radius-sm);
  font-size: 14px;
}

.search-input:focus {
  background-color: var(--surface);
}

.clear-btn {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted);
  font-size: 13px;
  padding: 4px 6px;
}

.clear-btn:hover {
  color: var(--text);
}

.cart-btn {
  display: inline-flex;
  align-items: center;
  gap: 11px;
  height: 44px;
  padding: 0 14px;
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  background-color: var(--surface);
  flex-shrink: 0;
}

.cart-btn:hover {
  border-color: var(--blue);
  background-color: var(--blue-light);
}

.cart-icon-box {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--navy);
}

.cart-badge {
  position: absolute;
  top: -7px;
  right: -9px;
  min-width: 18px;
  height: 18px;
  padding: 0 5px;
  border-radius: 99px;
  background-color: var(--text-muted);
  color: #ffffff;
  font-size: 11px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}

.cart-badge.has-items {
  background-color: var(--orange);
}

.cart-info {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  line-height: 1.2;
}

.cart-title {
  font-size: 11px;
  color: var(--text-secondary);
  font-weight: 500;
}

.cart-amount {
  font-size: 13px;
  font-weight: 700;
  color: var(--navy);
}

@media (max-width: 780px) {
  .header-inner {
    flex-wrap: wrap;
    height: auto;
    padding-top: 12px;
    padding-bottom: 12px;
    gap: 10px;
  }

  .logo-img {
    height: 36px;
  }

  .catalog-btn {
    height: 38px;
    padding: 0 12px;
    font-size: 13px;
  }

  .search-wrap {
    order: 4;
    max-width: 100%;
    flex-basis: 100%;
  }

  .cart-info {
    display: none;
  }

  .cart-btn {
    padding: 0 12px;
    height: 38px;
  }
}
</style>

<script setup>
import { ref } from 'vue'
import CartItem from './CartItem.vue'
import EmptyState from './EmptyState.vue'

defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  items: {
    type: Array,
    default: () => []
  },
  cartCount: {
    type: Number,
    default: 0
  },
  cartTotal: {
    type: Number,
    default: 0
  },
  discountAmount: {
    type: Number,
    default: 0
  },
  finalTotal: {
    type: Number,
    default: 0
  },
  appliedPromo: {
    type: String,
    default: ''
  },
  promoError: {
    type: String,
    default: ''
  }
})

const emit = defineEmits([
  'close',
  'updateQuantity',
  'removeItem',
  'clearCart',
  'applyPromo',
  'removePromo',
  'openCheckout'
])

const promoInput = ref('')

function formatCurrency(amount) {
  return new Intl.NumberFormat('uk-UA').format(amount) + ' ₴'
}

function handleApplyPromo() {
  if (promoInput.value.trim()) {
    emit('applyPromo', promoInput.value.trim())
    promoInput.value = ''
  }
}
</script>

<template>
  <Transition name="cart-drawer">
    <div v-if="isOpen" class="cart-overlay" @wheel.self.prevent>
      <div class="cart-backdrop" @click="emit('close')" @wheel.prevent></div>

      <aside class="cart-panel">
        <div class="cart-header" @wheel.prevent>
          <div class="cart-title-wrap">
            <h2 class="cart-title">Кошик покупок</h2>
            <span v-if="cartCount > 0" class="cart-qty">{{ cartCount }} шт.</span>
          </div>
          <button type="button" class="close-btn" @click="emit('close')">✕</button>
        </div>

        <div class="cart-body">
          <EmptyState
            v-if="items.length === 0"
            title="Кошик порожній"
            message="Додайте товари з каталогу, щоб оформити замовлення."
            action-text="До каталогу"
            @action="emit('close')"
          />

          <div v-else>
            <TransitionGroup name="cart-list" tag="div" class="items-list">
              <CartItem
                v-for="item in items"
                :key="item.id"
                :item="item"
                @update-quantity="emit('updateQuantity', $event)"
                @remove-item="emit('removeItem', $event)"
              />
            </TransitionGroup>

            <div class="clear-row">
              <button type="button" class="clear-btn" @click="emit('clearCart')">
                Очистити весь кошик
              </button>
            </div>

            <div class="promo-section">
              <label class="promo-label">Промокод на знижку</label>
              <div v-if="!appliedPromo" class="promo-row">
                <input
                  v-model="promoInput"
                  type="text"
                  placeholder="Введіть промокод"
                  class="promo-input"
                  @keyup.enter="handleApplyPromo"
                />
                <button type="button" class="promo-btn" @click="handleApplyPromo">
                  Застосувати
                </button>
              </div>

              <div v-else class="promo-active">
                <span>Промокод <strong>{{ appliedPromo }}</strong> активний</span>
                <button type="button" class="promo-remove" @click="emit('removePromo')">Скасувати</button>
              </div>

              <div v-if="promoError" class="promo-err">{{ promoError }}</div>
            </div>
          </div>
        </div>

        <div v-if="items.length > 0" class="cart-footer" @wheel.prevent>
          <div class="totals">
            <div class="total-line">
              <span>Вартість товарів:</span>
              <span>{{ formatCurrency(cartTotal) }}</span>
            </div>
            <div v-if="discountAmount > 0" class="total-line discount">
              <span>Знижка за промокодом:</span>
              <span>-{{ formatCurrency(discountAmount) }}</span>
            </div>
            <div class="total-line final">
              <span>Разом до сплати:</span>
              <span>{{ formatCurrency(finalTotal) }}</span>
            </div>
          </div>

          <button type="button" class="checkout-btn" @click="emit('openCheckout')">
            Оформити замовлення
          </button>
        </div>
      </aside>
    </div>
  </Transition>
</template>

<style scoped>
.cart-overlay {
  position: fixed;
  inset: 0;
  z-index: 50;
  display: flex;
  justify-content: flex-end;
}

.cart-backdrop {
  position: fixed;
  inset: 0;
  background-color: rgba(10, 20, 40, 0.45);
  backdrop-filter: blur(2px);
  touch-action: none;
}

.cart-panel {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 450px;
  background-color: var(--surface);
  height: 100%;
  display: flex;
  flex-direction: column;
  box-shadow: -8px 0 28px rgba(10, 20, 40, 0.14);
}

.cart-drawer-enter-active,
.cart-drawer-leave-active {
  transition: opacity 0.34s cubic-bezier(0.22, 1, 0.36, 1);
}

.cart-drawer-enter-active .cart-backdrop,
.cart-drawer-leave-active .cart-backdrop {
  transition: opacity 0.34s cubic-bezier(0.22, 1, 0.36, 1);
}

.cart-drawer-enter-active .cart-panel,
.cart-drawer-leave-active .cart-panel {
  transition: transform 0.36s cubic-bezier(0.22, 1, 0.36, 1);
}

.cart-drawer-enter-from,
.cart-drawer-leave-to {
  opacity: 0;
}

.cart-drawer-enter-from .cart-panel,
.cart-drawer-leave-to .cart-panel {
  transform: translateX(100%);
}

.cart-list-enter-active,
.cart-list-leave-active {
  transition: all 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.cart-list-enter-from,
.cart-list-leave-to {
  opacity: 0;
  transform: translateX(18px);
}

.cart-header {
  padding: 18px 20px;
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.cart-title-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
}

.cart-title {
  font-size: 18px;
  font-weight: 700;
  color: var(--navy);
}

.cart-qty {
  font-size: 12px;
  font-weight: 700;
  background-color: var(--blue-light);
  color: var(--blue);
  padding: 2px 8px;
  border-radius: 99px;
}

.close-btn {
  width: 32px;
  height: 32px;
  border-radius: 6px;
  font-size: 15px;
  color: var(--text-secondary);
  background-color: var(--bg);
}

.close-btn:hover {
  color: var(--navy);
  background-color: var(--border);
}

.cart-body {
  flex: 1;
  overflow-y: auto;
  overscroll-behavior: contain;
  padding: 16px 20px;
}

.clear-row {
  margin-top: 12px;
  text-align: right;
}

.clear-btn {
  font-size: 12px;
  color: var(--text-muted);
  text-decoration: underline;
}

.clear-btn:hover {
  color: var(--red);
}

.promo-section {
  margin-top: 18px;
  padding: 14px;
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
}

.promo-label {
  display: block;
  font-size: 12px;
  font-weight: 600;
  color: var(--navy);
  margin-bottom: 8px;
}

.promo-row {
  display: flex;
  gap: 8px;
}

.promo-input {
  flex: 1;
  height: 36px;
  padding: 0 10px;
  font-size: 13px;
}

.promo-btn {
  height: 36px;
  padding: 0 14px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
}

.promo-btn:hover {
  background-color: var(--blue);
}

.promo-active {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: var(--green-bg);
  color: var(--green);
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 13px;
}

.promo-remove {
  font-size: 12px;
  font-weight: 600;
  text-decoration: underline;
  color: var(--green);
}

.promo-err {
  margin-top: 6px;
  font-size: 12px;
  color: var(--red);
}

.cart-footer {
  padding: 18px 20px;
  border-top: 1px solid var(--border);
  background-color: var(--surface);
}

.totals {
  margin-bottom: 14px;
}

.total-line {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  color: var(--text-secondary);
  margin-bottom: 6px;
}

.total-line.discount {
  color: var(--green);
  font-weight: 600;
}

.total-line.final {
  font-size: 18px;
  font-weight: 800;
  color: var(--navy);
  margin-top: 8px;
  padding-top: 10px;
  border-top: 1px solid var(--border);
}

.checkout-btn {
  width: 100%;
  height: 44px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 14px;
  font-weight: 700;
}

.checkout-btn:hover {
  background-color: var(--blue);
  transform: translateY(-1px);
}
</style>

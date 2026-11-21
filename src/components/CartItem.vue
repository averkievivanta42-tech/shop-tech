<script setup>
const props = defineProps({
  item: {
    type: Object,
    required: true
  }
})

const emit = defineEmits(['updateQuantity', 'removeItem'])

function formatCurrency(amount) {
  return new Intl.NumberFormat('uk-UA').format(amount) + ' ₴'
}

function handleDecrease() {
  if (props.item.quantity > 1) {
    emit('updateQuantity', {
      id: props.item.id,
      quantity: props.item.quantity - 1
    })
  } else {
    emit('removeItem', props.item.id)
  }
}

function handleIncrease() {
  emit('updateQuantity', {
    id: props.item.id,
    quantity: props.item.quantity + 1
  })
}

function handleRemove() {
  emit('removeItem', props.item.id)
}

function handleImageError(e) {
  e.target.src = 'https://i.moyo.ua/img/products/7376/99_600.jpg'
}
</script>

<template>
  <div class="cart-item">
    <div class="item-img-box">
      <img
        :src="item.image"
        :alt="item.name"
        class="item-img"
        @error="handleImageError"
      />
    </div>

    <div class="item-info">
      <div class="item-title">{{ item.name }}</div>
      <div class="item-price">{{ formatCurrency(item.price) }} / шт.</div>
    </div>

    <div class="item-controls">
      <div class="qty-box">
        <button type="button" class="qty-btn" @click="handleDecrease">-</button>
        <span class="qty-num">{{ item.quantity }}</span>
        <button type="button" class="qty-btn" @click="handleIncrease">+</button>
      </div>

      <div class="item-sum">{{ formatCurrency(item.price * item.quantity) }}</div>

      <button type="button" class="remove-btn" @click="handleRemove" title="Видалити">
        ✕
      </button>
    </div>
  </div>
</template>

<style scoped>
.cart-item {
  display: grid;
  grid-template-columns: 60px 1fr auto;
  align-items: center;
  gap: 12px;
  padding: 12px 0;
  border-bottom: 1px solid var(--border);
}

.item-img-box {
  width: 60px;
  height: 60px;
  padding: 4px;
  background-color: #ffffff;
  border: 1px solid var(--border);
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.item-img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.item-info {
  min-width: 0;
}

.item-title {
  font-size: 13px;
  font-weight: 600;
  color: var(--navy);
  margin-bottom: 3px;
  line-height: 1.35;
}

.item-price {
  font-size: 12px;
  color: var(--text-muted);
}

.item-controls {
  display: flex;
  align-items: center;
  gap: 10px;
}

.qty-box {
  display: inline-flex;
  align-items: center;
  border: 1px solid var(--border);
  border-radius: 6px;
  background-color: var(--bg);
}

.qty-btn {
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 15px;
  font-weight: 600;
  color: var(--navy);
}

.qty-btn:hover {
  color: var(--blue);
}

.qty-num {
  min-width: 26px;
  text-align: center;
  font-size: 13px;
  font-weight: 700;
  color: var(--navy);
}

.item-sum {
  font-size: 14px;
  font-weight: 700;
  color: var(--navy);
  min-width: 82px;
  text-align: right;
}

.remove-btn {
  width: 28px;
  height: 28px;
  font-size: 12px;
  color: var(--text-muted);
  border-radius: 6px;
}

.remove-btn:hover {
  color: var(--red);
  background-color: var(--red-bg);
}

@media (max-width: 520px) {
  .cart-item {
    grid-template-columns: 52px 1fr;
  }

  .item-controls {
    grid-column: 1 / -1;
    justify-content: space-between;
  }

  .item-sum {
    text-align: left;
  }
}
</style>

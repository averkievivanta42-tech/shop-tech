<script setup>
import { ref, reactive } from 'vue'

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  items: {
    type: Array,
    default: () => []
  },
  finalTotal: {
    type: Number,
    default: 0
  },
  discountAmount: {
    type: Number,
    default: 0
  }
})

const emit = defineEmits(['close', 'orderSubmitted'])

const form = reactive({
  fullName: '',
  phone: '+380',
  email: '',
  city: '',
  deliveryMethod: 'nova-branch',
  deliveryAddress: '',
  paymentMethod: 'cod',
  comment: ''
})

const errors = reactive({
  fullName: '',
  phone: '',
  email: '',
  city: '',
  deliveryAddress: ''
})

const isSubmitting = ref(false)
const orderSuccess = ref(false)
const orderNumber = ref('')

function formatCurrency(amount) {
  return new Intl.NumberFormat('uk-UA').format(amount) + ' ₴'
}

function validateFullName() {
  const trimmed = form.fullName.trim()
  if (!trimmed) {
    errors.fullName = "Введіть прізвище та ім'я"
    return false
  }
  if (trimmed.length < 3) {
    errors.fullName = "Мінімум 3 символи"
    return false
  }
  const nameRegex = /^[A-Za-zА-Яа-яІіЇїЄєҐґ'\s-]+$/
  if (!nameRegex.test(trimmed)) {
    errors.fullName = "Тільки літери"
    return false
  }
  errors.fullName = ''
  return true
}

function validatePhone() {
  const trimmed = form.phone.trim()
  const cleanPhone = trimmed.replace(/[\s()-]/g, '')
  const phoneRegex = /^(\+38)?0\d{9}$/
  if (!trimmed) {
    errors.phone = 'Введіть номер телефону'
    return false
  }
  if (!phoneRegex.test(cleanPhone)) {
    errors.phone = 'Формат: +380991234567'
    return false
  }
  errors.phone = ''
  return true
}

function validateEmail() {
  const trimmed = form.email.trim()
  if (!trimmed) {
    errors.email = 'Введіть email'
    return false
  }
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(trimmed)) {
    errors.email = 'Некоректний формат email'
    return false
  }
  errors.email = ''
  return true
}

function validateCity() {
  if (!form.city.trim()) {
    errors.city = 'Вкажіть місто'
    return false
  }
  errors.city = ''
  return true
}

function validateAddress() {
  if (!form.deliveryAddress.trim()) {
    errors.deliveryAddress = 'Вкажіть відділення або адресу'
    return false
  }
  errors.deliveryAddress = ''
  return true
}

function validateAll() {
  const a = validateFullName()
  const b = validatePhone()
  const c = validateEmail()
  const d = validateCity()
  const e = validateAddress()
  return a && b && c && d && e
}

function handleSubmit() {
  if (!validateAll()) return

  isSubmitting.value = true

  setTimeout(() => {
    isSubmitting.value = false
    orderNumber.value = 'MT-' + Math.floor(10000 + Math.random() * 90000)
    orderSuccess.value = true

    emit('orderSubmitted', {
      orderNumber: orderNumber.value,
      customer: { ...form },
      items: [...props.items],
      finalTotal: props.finalTotal
    })
  }, 500)
}

function handleClose() {
  if (orderSuccess.value) {
    orderSuccess.value = false
    form.fullName = ''
    form.phone = '+380'
    form.email = ''
    form.city = ''
    form.deliveryAddress = ''
    form.comment = ''
  }
  emit('close')
}
</script>

<template>
  <Transition name="order-modal">
    <div v-if="isOpen" class="modal-wrap">
      <div class="modal-backdrop" @click="handleClose"></div>

      <div class="modal-box">
        <div class="modal-header">
          <h2 class="modal-title">
            {{ orderSuccess ? 'Замовлення прийнято' : 'Оформлення замовлення' }}
          </h2>
          <button type="button" class="close-btn" @click="handleClose">✕</button>
        </div>

        <div v-if="!orderSuccess" class="modal-body">
          <form @submit.prevent="handleSubmit" class="form" novalidate>
            <div class="field">
              <label class="label">Прізвище та ім'я *</label>
              <input
                v-model="form.fullName"
                type="text"
                placeholder="Іваненко Іван"
                class="input"
                :class="{ 'input-err': errors.fullName }"
                @blur="validateFullName"
              />
              <span v-if="errors.fullName" class="err-msg">{{ errors.fullName }}</span>
            </div>

            <div class="row-2">
              <div class="field">
                <label class="label">Телефон *</label>
                <input
                  v-model="form.phone"
                  type="tel"
                  placeholder="+380991234567"
                  class="input"
                  :class="{ 'input-err': errors.phone }"
                  @blur="validatePhone"
                />
                <span v-if="errors.phone" class="err-msg">{{ errors.phone }}</span>
              </div>

              <div class="field">
                <label class="label">Email *</label>
                <input
                  v-model="form.email"
                  type="email"
                  placeholder="name@example.com"
                  class="input"
                  :class="{ 'input-err': errors.email }"
                  @blur="validateEmail"
                />
                <span v-if="errors.email" class="err-msg">{{ errors.email }}</span>
              </div>
            </div>

            <div class="row-2">
              <div class="field">
                <label class="label">Місто *</label>
                <input
                  v-model="form.city"
                  type="text"
                  placeholder="Київ"
                  class="input"
                  :class="{ 'input-err': errors.city }"
                  @blur="validateCity"
                />
                <span v-if="errors.city" class="err-msg">{{ errors.city }}</span>
              </div>

              <div class="field">
                <label class="label">Спосіб доставки</label>
                <select v-model="form.deliveryMethod" class="input">
                  <option value="nova-branch">Нова Пошта (відділення)</option>
                  <option value="nova-locker">Нова Пошта (поштомат)</option>
                  <option value="pickup">Самовивіз</option>
                </select>
              </div>
            </div>

            <div class="field">
              <label class="label">Номер відділення або адреса *</label>
              <input
                v-model="form.deliveryAddress"
                type="text"
                placeholder="Відділення № 12"
                class="input"
                :class="{ 'input-err': errors.deliveryAddress }"
                @blur="validateAddress"
              />
              <span v-if="errors.deliveryAddress" class="err-msg">{{ errors.deliveryAddress }}</span>
            </div>

            <div class="field">
              <label class="label">Спосіб оплати</label>
              <select v-model="form.paymentMethod" class="input">
                <option value="cod">Оплата під час отримання</option>
                <option value="card">Оплата карткою онлайн</option>
                <option value="iban">Безготівковий розрахунок (ФОП Аверкієв І.О.)</option>
              </select>
            </div>

            <div class="field">
              <label class="label">Коментар до замовлення</label>
              <textarea
                v-model="form.comment"
                rows="2"
                class="input textarea"
              ></textarea>
            </div>

            <div class="order-total-bar">
              <span>Сума до сплати:</span>
              <strong>{{ formatCurrency(finalTotal) }}</strong>
            </div>

            <button type="submit" class="submit-btn" :disabled="isSubmitting">
              {{ isSubmitting ? 'Обробка...' : 'Підтвердити замовлення' }}
            </button>
          </form>
        </div>

        <div v-else class="modal-body success-view">
          <p class="success-num">Замовлення №{{ orderNumber }}</p>
          <p class="success-desc">
            Дякуємо, <strong>{{ form.fullName }}</strong>! Замовлення в магазині <strong>МегаТренд</strong> на суму
            <strong>{{ formatCurrency(finalTotal) }}</strong> успішно зареєстровано.
          </p>
          <button type="button" class="submit-btn" @click="handleClose">
            Повернутися до магазину
          </button>
        </div>
      </div>
    </div>
  </Transition>
</template>

<style scoped>
.modal-wrap {
  position: fixed;
  inset: 0;
  z-index: 60;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}

.modal-backdrop {
  position: fixed;
  inset: 0;
  background-color: rgba(10, 20, 40, 0.48);
  backdrop-filter: blur(3px);
}

.modal-box {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 520px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 16px 40px rgba(10, 20, 40, 0.2);
}

.order-modal-enter-active,
.order-modal-leave-active {
  transition: opacity 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.order-modal-enter-active .modal-box,
.order-modal-leave-active .modal-box {
  transition: transform 0.32s cubic-bezier(0.22, 1, 0.36, 1), opacity 0.3s cubic-bezier(0.22, 1, 0.36, 1);
}

.order-modal-enter-from,
.order-modal-leave-to {
  opacity: 0;
}

.order-modal-enter-from .modal-box,
.order-modal-leave-to .modal-box {
  transform: scale(0.95) translateY(10px);
  opacity: 0;
}

.modal-header {
  padding: 16px 20px;
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.modal-title {
  font-size: 18px;
  font-weight: 700;
  color: var(--navy);
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

.modal-body {
  padding: 20px;
  overflow-y: auto;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 13px;
}

.row-2 {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.label {
  font-size: 13px;
  font-weight: 600;
  color: var(--navy);
}

.input {
  height: 38px;
  padding: 0 12px;
  width: 100%;
}

.textarea {
  height: auto;
  padding: 8px 12px;
  resize: vertical;
}

.input-err {
  border-color: var(--red);
}

.err-msg {
  font-size: 12px;
  color: var(--red);
}

.order-total-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 13px 16px;
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  font-size: 16px;
  color: var(--navy);
  margin-top: 4px;
}

.submit-btn {
  height: 44px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 14px;
  font-weight: 700;
}

.submit-btn:hover:not(:disabled) {
  background-color: var(--blue);
  transform: translateY(-1px);
}

.success-view {
  text-align: center;
  padding: 32px 24px;
}

.success-num {
  font-size: 20px;
  font-weight: 800;
  color: var(--navy);
  margin-bottom: 10px;
}

.success-desc {
  font-size: 14px;
  color: var(--text-secondary);
  margin-bottom: 22px;
  line-height: 1.55;
}

@media (max-width: 520px) {
  .row-2 {
    grid-template-columns: 1fr;
  }
}
</style>

<script setup>
defineProps({
  message: {
    type: String,
    default: ''
  },
  type: {
    type: String,
    default: 'success'
  },
  visible: {
    type: Boolean,
    default: false
  }
})
</script>

<template>
  <Transition name="toast-pop">
    <div v-if="visible" class="toast" :class="'toast-' + type">
      {{ message }}
    </div>
  </Transition>
</template>

<style scoped>
.toast {
  position: fixed;
  bottom: 22px;
  right: 22px;
  z-index: 90;
  background-color: var(--navy);
  color: #ffffff;
  font-size: 13px;
  font-weight: 500;
  padding: 12px 18px;
  border-radius: var(--radius-sm);
  box-shadow: 0 8px 24px rgba(10, 20, 40, 0.22);
  border-left: 4px solid var(--blue);
}

.toast-error {
  border-left-color: var(--red);
}

.toast-pop-enter-active,
.toast-pop-leave-active {
  transition: opacity 0.28s cubic-bezier(0.22, 1, 0.36, 1), transform 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.toast-pop-enter-from,
.toast-pop-leave-to {
  opacity: 0;
  transform: translateY(14px) scale(0.97);
}

@media (max-width: 580px) {
  .toast {
    left: 16px;
    right: 16px;
    bottom: 16px;
    text-align: center;
  }
}
</style>

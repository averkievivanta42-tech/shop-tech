<script setup>
defineProps({
  title: {
    type: String,
    required: true
  },
  message: {
    type: String,
    required: true
  },
  actionText: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['action'])
</script>

<template>
  <div class="empty-box">
    <h3 class="empty-title">{{ title }}</h3>
    <p class="empty-msg">{{ message }}</p>
    <button
      v-if="actionText"
      type="button"
      class="empty-btn"
      @click="emit('action')"
    >
      {{ actionText }}
    </button>
  </div>
</template>

<style scoped>
.empty-box {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 44px 20px;
  text-align: center;
  margin: 16px 0;
}

.empty-title {
  font-size: 17px;
  font-weight: 700;
  margin-bottom: 6px;
  color: var(--navy);
}

.empty-msg {
  font-size: 14px;
  color: var(--text-secondary);
  max-width: 400px;
  margin: 0 auto 18px;
}

.empty-btn {
  height: 38px;
  padding: 0 18px;
  background-color: var(--navy);
  color: #ffffff;
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
}

.empty-btn:hover {
  background-color: var(--blue);
}
</style>

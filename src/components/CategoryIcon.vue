<script setup>
defineProps({
  name: {
    type: String,
    required: true
  }
})
</script>

<template>
  <svg
    class="cat-svg-icon"
    width="26"
    height="26"
    viewBox="0 0 24 24"
    fill="none"
    stroke="currentColor"
    stroke-width="1.75"
    stroke-linecap="round"
    stroke-linejoin="round"
  >
    <template v-if="name === 'Усі'">
      <rect x="3" y="3" width="7.5" height="7.5" rx="2" class="icon-tint" />
      <rect x="13.5" y="3" width="7.5" height="7.5" rx="2" />
      <rect x="3" y="13.5" width="7.5" height="7.5" rx="2" />
      <rect x="13.5" y="13.5" width="7.5" height="7.5" rx="2" class="icon-tint" />
    </template>

    <template v-else-if="name === 'Ноутбуки'">
      <rect x="3.5" y="4.5" width="17" height="11.5" rx="2" class="icon-tint" />
      <path d="M2 19h20" />
      <path d="M9.5 16h5" />
    </template>

    <template v-else-if="name === 'Смартфони'">
      <rect x="6.5" y="2.5" width="11" height="19" rx="2.5" class="icon-tint" />
      <line x1="10.5" y1="5.5" x2="13.5" y2="5.5" />
      <circle cx="12" cy="18.2" r="0.8" fill="currentColor" />
    </template>

    <template v-else-if="name === 'Планшети'">
      <rect x="4" y="3" width="16" height="18" rx="2.5" class="icon-tint" />
      <line x1="10.5" y1="18" x2="13.5" y2="18" />
      <path d="M14.5 7.5l-4 4" />
    </template>

    <template v-else-if="name === 'Аудіо'">
      <path d="M4 13v-1a8 8 0 0 1 16 0v1" />
      <rect x="3" y="13" width="4" height="6.5" rx="2" class="icon-tint" />
      <rect x="17" y="13" width="4" height="6.5" rx="2" class="icon-tint" />
    </template>

    <template v-else-if="name === 'Смарт-годинники'">
      <path d="M9 2.5h6l.7 3.5H8.3L9 2.5z" />
      <path d="M8.3 18h7.4L15 21.5H9L8.3 18z" />
      <rect x="6.5" y="6" width="11" height="12" rx="3" class="icon-tint" />
      <polyline points="12 9.5 12 12 13.8 13.2" />
    </template>

    <template v-else-if="name === 'Монітори'">
      <rect x="2.5" y="3.5" width="19" height="12.5" rx="2" class="icon-tint" />
      <line x1="8.5" y1="20.5" x2="15.5" y2="20.5" />
      <line x1="12" y1="16" x2="12" y2="20.5" />
    </template>

    <template v-else-if="name === 'Периферія'">
      <rect x="2.5" y="5.5" width="19" height="13" rx="2.2" class="icon-tint" />
      <line x1="6" y1="9.5" x2="6.01" y2="9.5" />
      <line x1="10" y1="9.5" x2="10.01" y2="9.5" />
      <line x1="14" y1="9.5" x2="14.01" y2="9.5" />
      <line x1="18" y1="9.5" x2="18.01" y2="9.5" />
      <line x1="6" y1="12.5" x2="6.01" y2="12.5" />
      <line x1="10" y1="12.5" x2="10.01" y2="12.5" />
      <line x1="14" y1="12.5" x2="14.01" y2="12.5" />
      <line x1="18" y1="12.5" x2="18.01" y2="12.5" />
      <line x1="8" y1="15.5" x2="16" y2="15.5" />
    </template>

    <template v-else>
      <rect x="6" y="4.5" width="12" height="16.5" rx="2.2" class="icon-tint" />
      <line x1="9.5" y1="2.5" x2="14.5" y2="2.5" />
      <polygon points="12.5 9 9.8 13 12.2 13 11.5 16.5 14.2 12.5 11.8 12.5 12.5 9" fill="currentColor" stroke="none" />
    </template>
  </svg>
</template>

<style scoped>
.cat-svg-icon {
  display: block;
  flex-shrink: 0;
  transition: transform 0.28s cubic-bezier(0.22, 1, 0.36, 1), color 0.24s ease;
}

.icon-tint {
  fill: currentColor;
  fill-opacity: 0.1;
  transition: fill-opacity 0.24s ease;
}
</style>

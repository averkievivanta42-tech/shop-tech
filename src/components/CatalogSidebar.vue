<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  minBound: {
    type: Number,
    default: 0
  },
  maxBound: {
    type: Number,
    default: 100000
  },
  priceMin: {
    type: Number,
    default: 0
  },
  priceMax: {
    type: Number,
    default: 100000
  },
  availableBrands: {
    type: Array,
    default: () => []
  },
  selectedBrands: {
    type: Array,
    default: () => []
  },
  attributeGroups: {
    type: Array,
    default: () => []
  },
  selectedAttributes: {
    type: Object,
    default: () => ({})
  },
  onlyDiscounted: {
    type: Boolean,
    default: false
  },
  onlyInStock: {
    type: Boolean,
    default: false
  },
  activeFiltersCount: {
    type: Number,
    default: 0
  },
  filteredCount: {
    type: Number,
    default: 0
  },
  isMobileOpen: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits([
  'updatePrice',
  'toggleBrand',
  'toggleAttribute',
  'update:onlyDiscounted',
  'update:onlyInStock',
  'resetAll',
  'closeMobile'
])

const localMin = ref(props.priceMin)
const localMax = ref(props.priceMax)
const collapsedGroups = ref({})

watch(
  () => [props.priceMin, props.priceMax],
  ([newMin, newMax]) => {
    localMin.value = newMin
    localMax.value = newMax
  }
)

function applyPriceRange() {
  let minVal = Number(localMin.value)
  let maxVal = Number(localMax.value)
  if (isNaN(minVal) || minVal < 0) minVal = props.minBound
  if (isNaN(maxVal) || maxVal <= 0) maxVal = props.maxBound
  if (minVal > maxVal) {
    const temp = minVal
    minVal = maxVal
    maxVal = temp
  }
  localMin.value = minVal
  localMax.value = maxVal
  emit('updatePrice', { min: minVal, max: maxVal })
}

function toggleGroupCollapse(groupKey) {
  collapsedGroups.value[groupKey] = !collapsedGroups.value[groupKey]
}

function isAttrChecked(groupName, optionValue) {
  const arr = props.selectedAttributes[groupName]
  return Array.isArray(arr) && arr.includes(optionValue)
}
</script>

<template>
  <div class="sidebar-wrapper" :class="{ 'mobile-open': isMobileOpen }">
    <div v-if="isMobileOpen" class="sidebar-backdrop" @click="emit('closeMobile')"></div>

    <aside class="filters-sidebar">
      <div class="sidebar-mobile-header">
        <h3 class="sidebar-mobile-title">Фільтри підбору</h3>
        <button type="button" class="sidebar-close-btn" @click="emit('closeMobile')">✕</button>
      </div>

      <div v-if="activeFiltersCount > 0" class="reset-top-bar">
        <span class="active-count-text">Обрано фільтрів: <strong>{{ activeFiltersCount }}</strong></span>
        <button type="button" class="reset-link-btn" @click="emit('resetAll')">
          Скинути всі
        </button>
      </div>

      <div class="filter-block">
        <div class="filter-head" @click="toggleGroupCollapse('price')">
          <span class="filter-title">Ціна, ₴</span>
          <span class="chevron">{{ collapsedGroups['price'] ? '+' : '-' }}</span>
        </div>

        <div v-if="!collapsedGroups['price']" class="filter-body">
          <div class="price-inputs-row">
            <input
              v-model.number="localMin"
              type="number"
              class="price-input"
              :min="minBound"
              :max="maxBound"
              placeholder="Від"
              @keyup.enter="applyPriceRange"
            />
            <span class="price-sep">-</span>
            <input
              v-model.number="localMax"
              type="number"
              class="price-input"
              :min="minBound"
              :max="maxBound"
              placeholder="До"
              @keyup.enter="applyPriceRange"
            />
            <button type="button" class="price-ok-btn" @click="applyPriceRange">
              ОК
            </button>
          </div>
        </div>
      </div>

      <div class="filter-block">
        <div class="filter-head" @click="toggleGroupCollapse('status')">
          <span class="filter-title">Акції та наявність</span>
          <span class="chevron">{{ collapsedGroups['status'] ? '+' : '-' }}</span>
        </div>

        <div v-if="!collapsedGroups['status']" class="filter-body check-list">
          <label class="check-item">
            <input
              type="checkbox"
              :checked="onlyDiscounted"
              @change="emit('update:onlyDiscounted', $event.target.checked)"
            />
            <span class="check-label">Акційні пропозиції (зі знижкою)</span>
          </label>

          <label class="check-item">
            <input
              type="checkbox"
              :checked="onlyInStock"
              @change="emit('update:onlyInStock', $event.target.checked)"
            />
            <span class="check-label">Тільки в наявності</span>
          </label>
        </div>
      </div>

      <div v-if="availableBrands.length > 0" class="filter-block">
        <div class="filter-head" @click="toggleGroupCollapse('brand')">
          <span class="filter-title">Бренд</span>
          <span class="chevron">{{ collapsedGroups['brand'] ? '+' : '-' }}</span>
        </div>

        <div v-if="!collapsedGroups['brand']" class="filter-body check-list">
          <label
            v-for="brand in availableBrands"
            :key="brand.name"
            class="check-item"
          >
            <input
              type="checkbox"
              :checked="selectedBrands.includes(brand.name)"
              @change="emit('toggleBrand', brand.name)"
            />
            <span class="check-label">{{ brand.name }}</span>
            <span class="check-count">({{ brand.count }})</span>
          </label>
        </div>
      </div>

      <div
        v-for="group in attributeGroups"
        :key="group.name"
        class="filter-block"
      >
        <div class="filter-head" @click="toggleGroupCollapse(group.name)">
          <span class="filter-title">{{ group.name }}</span>
          <span class="chevron">{{ collapsedGroups[group.name] ? '+' : '-' }}</span>
        </div>

        <div v-if="!collapsedGroups[group.name]" class="filter-body check-list">
          <label
            v-for="opt in group.options"
            :key="opt.value"
            class="check-item"
          >
            <input
              type="checkbox"
              :checked="isAttrChecked(group.name, opt.value)"
              @change="emit('toggleAttribute', { group: group.name, value: opt.value })"
            />
            <span class="check-label">{{ opt.value }}</span>
            <span class="check-count">({{ opt.count }})</span>
          </label>
        </div>
      </div>

      <div class="sidebar-mobile-footer">
        <button type="button" class="apply-mobile-btn" @click="emit('closeMobile')">
          Показати товари ({{ filteredCount }})
        </button>
      </div>
    </aside>
  </div>
</template>

<style scoped>
.sidebar-wrapper {
  width: 270px;
  flex-shrink: 0;
}

.filters-sidebar {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  box-shadow: var(--shadow-sm);
  overflow: hidden;
}

.sidebar-mobile-header,
.sidebar-mobile-footer,
.sidebar-backdrop {
  display: none;
}

.reset-top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  background-color: var(--blue-light);
  border-bottom: 1px solid var(--border);
  font-size: 12px;
}

.active-count-text {
  color: var(--navy);
}

.reset-link-btn {
  color: var(--red);
  font-weight: 700;
  text-decoration: underline;
  font-size: 12px;
}

.filter-block {
  border-bottom: 1px solid var(--border);
}

.filter-block:last-of-type {
  border-bottom: none;
}

.filter-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 13px 16px;
  cursor: pointer;
  user-select: none;
}

.filter-head:hover {
  background-color: var(--bg);
}

.filter-title {
  font-size: 14px;
  font-weight: 700;
  color: var(--navy);
}

.chevron {
  font-size: 16px;
  font-weight: 700;
  color: var(--text-muted);
  line-height: 1;
}

.filter-body {
  padding: 2px 16px 14px;
}

.price-inputs-row {
  display: flex;
  align-items: center;
  gap: 6px;
}

.price-input {
  width: 100%;
  min-width: 0;
  height: 34px;
  padding: 0 8px;
  font-size: 13px;
  font-weight: 600;
}

.price-sep {
  color: var(--text-muted);
  font-weight: 600;
}

.price-ok-btn {
  height: 34px;
  padding: 0 12px;
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: 6px;
  font-size: 12px;
  font-weight: 700;
  color: var(--navy);
  flex-shrink: 0;
}

.price-ok-btn:hover {
  background-color: var(--navy);
  border-color: var(--navy);
  color: #ffffff;
}

.check-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  max-height: 230px;
  overflow-y: auto;
}

.check-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  color: var(--text);
  cursor: pointer;
  user-select: none;
}

.check-item:hover .check-label {
  color: var(--blue);
}

.check-item input {
  width: 16px;
  height: 16px;
  accent-color: var(--blue);
  cursor: pointer;
  flex-shrink: 0;
}

.check-label {
  flex: 1;
  line-height: 1.3;
}

.check-count {
  font-size: 12px;
  color: var(--text-muted);
}

@media (max-width: 920px) {
  .sidebar-wrapper {
    width: 0;
    position: fixed;
    z-index: 70;
    pointer-events: none;
  }

  .sidebar-wrapper.mobile-open {
    inset: 0;
    width: 100%;
    pointer-events: auto;
    display: flex;
  }

  .sidebar-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    background-color: rgba(10, 20, 40, 0.5);
  }

  .filters-sidebar {
    position: relative;
    z-index: 1;
    width: 100%;
    max-width: 340px;
    height: 100%;
    border-radius: 0;
    border: none;
    display: flex;
    flex-direction: column;
    overflow-y: auto;
    transform: translateX(-100%);
    transition: transform 0.22s ease;
  }

  .sidebar-wrapper.mobile-open .filters-sidebar {
    transform: translateX(0);
  }

  .sidebar-mobile-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 16px;
    border-bottom: 1px solid var(--border);
    background-color: var(--surface);
    position: sticky;
    top: 0;
    z-index: 2;
  }

  .sidebar-mobile-title {
    font-size: 17px;
    font-weight: 700;
    color: var(--navy);
  }

  .sidebar-close-btn {
    width: 32px;
    height: 32px;
    border-radius: 6px;
    background-color: var(--bg);
    color: var(--navy);
    font-size: 15px;
    font-weight: 700;
  }

  .sidebar-mobile-footer {
    display: block;
    padding: 14px 16px;
    border-top: 1px solid var(--border);
    background-color: var(--surface);
    position: sticky;
    bottom: 0;
    margin-top: auto;
    z-index: 2;
  }

  .apply-mobile-btn {
    width: 100%;
    height: 42px;
    background-color: var(--navy);
    color: #ffffff;
    border-radius: var(--radius-sm);
    font-size: 14px;
    font-weight: 700;
  }
}
</style>

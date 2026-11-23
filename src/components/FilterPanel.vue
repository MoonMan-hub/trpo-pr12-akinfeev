<template>
  <section class="panel filter-panel" aria-labelledby="filter-title">
    <div>
      <p class="eyebrow">Фильтрация</p>
      <h2 id="filter-title">Состояние задач</h2>
    </div>

    <div class="filter-list" role="group" aria-label="Фильтр по статусу">
      <button
        v-for="filter in filters"
        :key="filter.id"
        type="button"
        class="filter-button"
        :class="{ active: filter.id === value }"
        @click="$emit('update:value', filter.id)"
      >
        {{ filter.title }}
      </button>
    </div>
  </section>
</template>

<script>
export default {
  name: 'FilterPanel',
  props: {
    filters: {
      type: Array,
      required: true
    },
    value: {
      type: String,
      required: true
    }
  },
  emits: ['update:value']
}
</script>

<template>
  <section class="panel list-panel" aria-labelledby="list-title">
    <div class="section-heading">
      <div>
        <p class="eyebrow">Данные</p>
        <h2 id="list-title">Список задач</h2>
      </div>
      <span class="count">{{ items.length }}</span>
    </div>

    <div v-if="items.length" class="task-list">
      <ItemCard
        v-for="item in items"
        :key="item.id"
        :item="item"
        @select="$emit('select', $event)"
      />
    </div>

    <div v-else class="empty-state">
      <strong>Задач нет</strong>
      <p>Попробуйте выбрать другой фильтр или создать новую задачу.</p>
    </div>
  </section>
</template>

<script>
import ItemCard from './ItemCard.vue'

export default {
  name: 'ItemList',
  components: { ItemCard },
  props: {
    items: {
      type: Array,
      required: true
    }
  },
  emits: ['select']
}
</script>

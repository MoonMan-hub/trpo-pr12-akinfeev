<template>
  <article class="task-card">
    <div class="task-main">
      <div class="task-id">#{{ item.id }}</div>
      <div>
        <h3>{{ item.title }}</h3>
        <p class="task-meta">Приоритет: {{ priorityLabel }}</p>
      </div>
    </div>

    <div class="task-actions">
      <span class="status-badge" :class="`status-${item.status}`">{{ statusLabel }}</span>
      <button type="button" class="text-button" @click="$emit('select', item.id)">
        Выбрать
      </button>
    </div>
  </article>
</template>

<script>
export default {
  name: 'ItemCard',
  props: {
    item: {
      type: Object,
      required: true
    }
  },
  emits: ['select'],
  computed: {
    statusLabel() {
      return {
        new: 'Новая',
        doing: 'В работе',
        done: 'Готово'
      }[this.item.status] ?? this.item.status
    },
    priorityLabel() {
      return {
        1: 'низкий',
        2: 'средний',
        3: 'высокий'
      }[this.item.priority] ?? this.item.priority
    }
  }
}
</script>

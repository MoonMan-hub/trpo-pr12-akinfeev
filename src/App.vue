<template>
  <main class="page-shell">
    <header class="hero panel">
      <div>
        <p class="eyebrow">Практическая работа №12</p>
        <h1>Менеджер задач</h1>
        <p class="lead">
          Vue 3 · Options API · компоненты · props · emits · computed · состояния интерфейса
        </p>
      </div>
      <div class="hero-status" :class="`state-${uiState}`">
        <span class="state-dot"></span>
        {{ stateLabel }}
      </div>
    </header>

    <section class="stats-grid" aria-label="Сводка">
      <div class="stat-card">
        <span>Всего</span>
        <strong>{{ items.length }}</strong>
      </div>
      <div class="stat-card">
        <span>В работе</span>
        <strong>{{ doingCount }}</strong>
      </div>
      <div class="stat-card">
        <span>Готово</span>
        <strong>{{ doneCount }}</strong>
      </div>
      <div class="stat-card">
        <span>Показано</span>
        <strong>{{ visibleItems.length }}</strong>
      </div>
    </section>

    <div class="content-grid">
      <div class="main-column">
        <FilterPanel
          :filters="filters"
          :value="filter"
          @update:value="filter = $event"
        />

        <section class="panel controls-panel">
          <div class="search-wrap">
            <label for="search">Поиск</label>
            <input
              id="search"
              v-model.trim="query"
              type="search"
              placeholder="Введите название задачи"
            />
          </div>
          <button type="button" class="secondary-button" @click="simulateError">
            Проверить error
          </button>
        </section>

        <div v-if="uiState === 'loading'" class="panel state-panel">
          <div class="loader" aria-hidden="true"></div>
          <h2>Загрузка данных…</h2>
          <p>Приложение подготавливает dataset.</p>
        </div>

        <div v-else-if="uiState === 'error'" class="panel state-panel error-state">
          <div class="state-icon">!</div>
          <h2>Не удалось загрузить данные</h2>
          <p>{{ errorMessage }}</p>
          <button type="button" class="primary-button" @click="loadData">Повторить</button>
        </div>

        <ItemList
          v-else
          :items="visibleItems"
          @select="selectItem"
        />
      </div>

      <aside class="side-column">
        <section class="panel form-panel">
          <p class="eyebrow">Создание</p>
          <h2>Новая задача</h2>
          <form @submit.prevent="addItem" novalidate>
            <label for="title">Название</label>
            <input
              id="title"
              v-model.trim="form.title"
              type="text"
              placeholder="Например, Проверить отчёт"
              :aria-invalid="Boolean(formError)"
            />
            <p v-if="formError" class="field-error">{{ formError }}</p>

            <label for="status">Статус</label>
            <select id="status" v-model="form.status">
              <option value="new">Новая</option>
              <option value="doing">В работе</option>
              <option value="done">Готово</option>
            </select>

            <label for="priority">Приоритет</label>
            <select id="priority" v-model.number="form.priority">
              <option :value="1">Низкий</option>
              <option :value="2">Средний</option>
              <option :value="3">Высокий</option>
            </select>

            <button class="primary-button" type="submit">Добавить задачу</button>
          </form>
        </section>

        <section class="panel selected-panel">
          <p class="eyebrow">Выбор</p>
          <h2>Выбранная задача</h2>
          <div v-if="selectedItem" class="selected-content">
            <strong>{{ selectedItem.title }}</strong>
            <span>#{{ selectedItem.id }} · {{ selectedStatusLabel }}</span>
          </div>
          <p v-else class="muted">Нажмите «Выбрать» у любой задачи.</p>
        </section>
      </aside>
    </div>
  </main>
</template>

<script>
import itemsData from './data/items.json'
import filtersData from './data/filters.json'
import uiStatesData from './data/ui_states.json'
import FilterPanel from './components/FilterPanel.vue'
import ItemList from './components/ItemList.vue'

export default {
  name: 'App',
  components: {
    FilterPanel,
    ItemList
  },
  data() {
    return {
      items: [],
      filters: filtersData,
      uiStates: uiStatesData,
      filter: 'all',
      query: '',
      uiState: 'loading',
      errorMessage: '',
      selectedItemId: null,
      form: {
        title: '',
        status: 'new',
        priority: 1
      },
      formError: ''
    }
  },
  computed: {
    visibleItems() {
      const normalizedQuery = this.query.toLowerCase()

      return this.items.filter((item) => {
        const matchesFilter = this.filter === 'all' || item.status === this.filter
        const matchesQuery = item.title.toLowerCase().includes(normalizedQuery)
        return matchesFilter && matchesQuery
      })
    },
    doingCount() {
      return this.items.filter((item) => item.status === 'doing').length
    },
    doneCount() {
      return this.items.filter((item) => item.status === 'done').length
    },
    selectedItem() {
      return this.items.find((item) => item.id === this.selectedItemId) ?? null
    },
    selectedStatusLabel() {
      return this.selectedItem
        ? this.filters.find((filter) => filter.id === this.selectedItem.status)?.title ?? this.selectedItem.status
        : ''
    },
    stateLabel() {
      return {
        loading: 'Загрузка',
        error: 'Ошибка',
        empty: 'Пусто',
        success: 'Готово'
      }[this.uiState] ?? this.uiState
    }
  },
  mounted() {
    this.loadData()
  },
  methods: {
    loadData() {
      this.uiState = 'loading'
      this.errorMessage = ''

      window.setTimeout(() => {
        this.items = itemsData.map((item) => ({ ...item }))
        this.uiState = this.items.length ? 'success' : 'empty'
      }, 350)
    },
    simulateError() {
      this.uiState = 'error'
      this.errorMessage = this.uiStates.find((state) => state.state === 'error')?.message ?? 'Ошибка загрузки.'
    },
    selectItem(id) {
      this.selectedItemId = id
    },
    addItem() {
      if (!this.form.title) {
        this.formError = 'Введите название задачи.'
        return
      }

      this.formError = ''
      const nextId = this.items.length
        ? Math.max(...this.items.map((item) => item.id)) + 1
        : 1

      this.items.push({
        id: nextId,
        title: this.form.title,
        status: this.form.status,
        priority: this.form.priority
      })

      this.selectedItemId = nextId
      this.form.title = ''
      this.form.status = 'new'
      this.form.priority = 1
      this.uiState = 'success'
    }
  }
}
</script>

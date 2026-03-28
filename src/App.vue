<script setup>
import { computed, ref } from 'vue'
import { events } from './data/events'

const monthNames = [
  'Январь',
  'Февраль',
  'Март',
  'Апрель',
  'Май',
  'Июнь',
  'Июль',
  'Август',
  'Сентябрь',
  'Октябрь',
  'Ноябрь',
  'Декабрь',
]

const weekdayNames = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс']

const normalizedEvents = events
  .map((event, index) => ({
    ...event,
    id: `${event.date}-${index}`,
    dateObj: new Date(`${event.date}T12:00:00`),
  }))
  .sort((a, b) => a.dateObj - b.dateObj)

const currentMonth = ref(new Date(2026, 2, 1))
const selectedDate = ref(normalizedEvents.at(-1)?.date ?? formatDateKey(new Date()))

const eventsByDate = computed(() => {
  return normalizedEvents.reduce((acc, event) => {
    if (!acc[event.date]) {
      acc[event.date] = []
    }
    acc[event.date].push(event)
    return acc
  }, {})
})

const monthLabel = computed(() => {
  return `${monthNames[currentMonth.value.getMonth()]} ${currentMonth.value.getFullYear()}`
})

const monthGrid = computed(() => {
  const start = new Date(
    currentMonth.value.getFullYear(),
    currentMonth.value.getMonth(),
    1,
  )
  const end = new Date(
    currentMonth.value.getFullYear(),
    currentMonth.value.getMonth() + 1,
    0,
  )

  const startOffset = (start.getDay() + 6) % 7
  const cells = []

  for (let index = 0; index < startOffset; index += 1) {
    const fillerDate = new Date(start)
    fillerDate.setDate(start.getDate() - (startOffset - index))
    cells.push(buildCell(fillerDate, false))
  }

  for (let day = 1; day <= end.getDate(); day += 1) {
    const date = new Date(
      currentMonth.value.getFullYear(),
      currentMonth.value.getMonth(),
      day,
    )
    cells.push(buildCell(date, true))
  }

  const remainder = cells.length % 7
  if (remainder !== 0) {
    const tail = 7 - remainder
    for (let index = 1; index <= tail; index += 1) {
      const fillerDate = new Date(end)
      fillerDate.setDate(end.getDate() + index)
      cells.push(buildCell(fillerDate, false))
    }
  }

  return cells
})

const selectedEvents = computed(() => {
  return eventsByDate.value[selectedDate.value] ?? []
})

const selectedDateLabel = computed(() => {
  return formatLongDate(selectedDate.value)
})

function buildCell(date, inCurrentMonth) {
  const key = formatDateKey(date)

  return {
    key,
    day: date.getDate(),
    inCurrentMonth,
    isSelected: key === selectedDate.value,
    isToday: key === formatDateKey(new Date()),
    hasEvents: Boolean(eventsByDate.value[key]?.length),
  }
}

function previousMonth() {
  currentMonth.value = new Date(
    currentMonth.value.getFullYear(),
    currentMonth.value.getMonth() - 1,
    1,
  )
}

function nextMonth() {
  currentMonth.value = new Date(
    currentMonth.value.getFullYear(),
    currentMonth.value.getMonth() + 1,
    1,
  )
}

function selectDate(dateKey) {
  selectedDate.value = dateKey
}

function formatDateKey(date) {
  return date.toLocaleDateString('sv-SE')
}

function formatLongDate(dateKey) {
  const date = new Date(`${dateKey}T12:00:00`)
  return new Intl.DateTimeFormat('ru-RU', {
    day: 'numeric',
    month: 'long',
    year: 'numeric',
    weekday: 'long',
  }).format(date)
}
</script>

<template>
  <main class="app-shell">
    <section class="hero-card">
      <p class="eyebrow">Личный календарь</p>
      <div class="hero-copy">
        <h1>Наши встречи.</h1>
        <p>
          Квадраты показывают дни месяца. Нажимаешь на день и сразу видишь, что
          у вас там было.
        </p>
      </div>
    </section>

    <section class="calendar-layout">
      <div class="calendar-card">
        <div class="calendar-toolbar">
          <button class="nav-button" type="button" @click="previousMonth">
            ←
          </button>
          <div>
            <p class="toolbar-label">Месяц</p>
            <h2>{{ monthLabel }}</h2>
          </div>
          <button class="nav-button" type="button" @click="nextMonth">→</button>
        </div>

        <div class="weekday-row">
          <span v-for="weekday in weekdayNames" :key="weekday">
            {{ weekday }}
          </span>
        </div>

        <div class="calendar-grid">
          <button
            v-for="cell in monthGrid"
            :key="cell.key"
            class="day-cell"
            :class="{
              'is-muted': !cell.inCurrentMonth,
              'is-selected': cell.isSelected,
              'is-today': cell.isToday,
              'has-events': cell.hasEvents,
            }"
            type="button"
            @click="selectDate(cell.key)"
          >
            <span>{{ cell.day }}</span>
            <small v-if="cell.hasEvents"></small>
          </button>
        </div>
      </div>

      <aside class="details-card">
        <p class="eyebrow">День</p>
        <h3>{{ selectedDateLabel }}</h3>

        <div v-if="selectedEvents.length" class="event-list">
          <article v-for="event in selectedEvents" :key="event.id" class="event-item">
            <p class="event-date">{{ event.date }}</p>
            <p class="event-description">{{ event.description }}</p>
          </article>
        </div>

        <div v-else class="empty-state">
          <p>На этот день пока ничего не записано.</p>
          <p>Можно оставить его свободным или потом добавить встречу.</p>
        </div>
      </aside>
    </section>
  </main>
</template>

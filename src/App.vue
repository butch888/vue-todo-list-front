<script setup>
import { computed, reactive, ref, nextTick } from 'vue'

const todos = reactive([])
const todoListRef = ref(null)
const filter = ref('')

const removeTodo = (id) => {
  todos.splice(
    todos.findIndex((todo) => todo.id === id),
    1,
  )
}

const rest = computed(() => todos.filter((todo) => !todo.completed).length)

const clearCompleted = () => {
  const newTodos = []
  for (const todo of todos) {
    if (!todo.completed) {
      newTodos.push(todo)
    }
  }
  todos.length = 0
  todos.push(...newTodos)
}

const clearAll = () => {
  todos.splice(0, todos.length)
  filter.value = ''
}

const newTask = ref('')
const onSubmit = async (e) => {
  e.preventDefault()
  if (newTask.value === '') return

  todos.push({
    id: Date.now(),
    text: newTask.value,
    completed: false,
  })

  newTask.value = ''

  await nextTick()
  scrollToLastTodo()
  getAllTasks()
}

const scrollToLastTodo = () => {
  if (todoListRef.value && todoListRef.value.children.length > 0) {
    const lastTodo = todoListRef.value.children[todoListRef.value.children.length - 1]
    lastTodo.scrollIntoView({ behavior: 'smooth', block: 'nearest' })
  }
}

const getAllTasks = () => {
  filter.value = 'all'
}

const getActiveTasks = () => {
  filter.value = 'active'
}

const getCompletedTasks = () => {
  filter.value = 'completed'
}

const filteredTodos = computed(() => {
  switch (filter.value) {
    case 'active':
      return todos.filter((todo) => !todo.completed)
    case 'completed':
      return todos.filter((todo) => todo.completed)
    default:
      return todos
  }
})

// Добавляем computed свойство для проверки пустоты списка
const isListEmpty = computed(() => todos.length === 0)
</script>

<template>
  <div class="container todo-app">
    <h1 class="title">Todo List</h1>

    <form class="todo-form" v-on:submit="onSubmit">
      <input
        type="text"
        v-model="newTask"
        placeholder="Добавить новую задачу..."
        class="todo-form__input"
      />
      <button class="todo-form__button" :disabled="newTask.trim() === ''">Добавить</button>
    </form>

    <div class="btns">
      <button
        v-on:click="getAllTasks"
        :class="{ 'btn--active': filter === 'all' }"
        class="filter-btn"
        :disabled="isListEmpty"
      >
        Все
      </button>
      <button
        v-on:click="getActiveTasks"
        :class="{ 'btn--active': filter === 'active' }"
        class="filter-btn"
        :disabled="isListEmpty"
      >
        Активные
      </button>
      <button
        v-on:click="getCompletedTasks"
        :class="{ 'btn--active': filter === 'completed' }"
        class="filter-btn"
        :disabled="isListEmpty"
      >
        Завершенные
      </button>
    </div>

    <div class="todo-app__main">
      <ul class="todo-list" ref="todoListRef">
        <li
          v-for="todo in filteredTodos"
          :key="todo.id"
          class="todo-list__item"
          :class="{ 'todo-list__item--completed': todo.completed }"
        >
          <span class="todo-list__item-text">{{ todo.text }}</span>
          <button
            class="btn btn--check"
            aria-label="Завершить"
            v-on:click="todo.completed = !todo.completed"
            v-bind:disabled="todo.completed"
          >
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" width="15" height="22">
              <path
                d="M438.6 109.4c-12.5-12.5-32.8-12.5-45.3 0L160 320.7l-92.3-92.3c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3l112 112c12.5 12.5 32.8 12.5 45.3 0l288-288c12.6-12.5 12.6-32.8 .1-45.3z"
              />
            </svg>
          </button>
          <button
            class="btn btn--delete"
            aria-label="Удалить"
            v-on:click="removeTodo(todo.id)"
            :disabled="isListEmpty"
          >
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" width="15" height="22">
              <path
                d="M135.2 17.7L128 32 32 32C14.3 32 0 46.3 0 64S14.3 96 32 96l384 0c17.7 0 32-14.3 32-32s-14.3-32-32-32l-96 0-7.2-14.3C307.4 6.8 296.3 0 284.2 0L163.8 0c-12.1 0-23.2 6.8-28.6 17.7zM416 128L32 128 53.2 467c1.6 25.3 22.6 45 47.9 45l245.8 0c25.3 0 46.3-19.7 47.9-45L416 128z"
              />
            </svg>
          </button>
        </li>
      </ul>
      <div
        v-if="
          todos.length === 0 ||
          (todos.filter((todo) => !todo.completed).length === 0 && filter === 'active') ||
          (todos.filter((todo) => todo.completed).length === 0 && filter === 'completed')
        "
        class="todo-list__empty"
      >
        <p>Список задач пуст</p>
      </div>
    </div>

    <p class="todo-app__footer-text">Невыполненные задания: {{ rest }}</p>

    <div class="todo-app__footer">
      <button class="btn btn--clear" v-on:click="clearCompleted" :disabled="isListEmpty">
        Удалить завершенные
      </button>
      <button class="btn btn--clear" v-on:click="clearAll" :disabled="isListEmpty">
        Очистить список
      </button>
    </div>
  </div>
</template>

<style scoped>
.container {
  margin: 20px auto;
  max-width: 500px;
  padding: 0 15px;
}

.title {
  text-align: center;
  color: #2d3748;
  margin-bottom: 2rem;
  font-size: 2.5rem;
  font-weight: 300;
}

.todo-form {
  display: flex;
  align-items: stretch;
  gap: 0.5rem;
  margin: 2rem auto;
  max-width: 500px;
  background: white;
  border-radius: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.todo-form__input {
  flex-grow: 1;
  padding: 0.75rem 1rem;
  border: none;
  outline: none;
  font-size: 1rem;
  background: #f7fafc;
  transition: background-color 0.2s ease;
}

.todo-form__input:focus {
  background: white;
  box-shadow: inset 0 0 0 2px #6574cd;
}

.todo-form__input::placeholder {
  color: #a0aec0;
}

.todo-form__button {
  padding: 0.75rem 1.5rem;
  border: none;
  background: #6574cd;
  color: white;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.todo-form__button:hover {
  background: #5661b3;
  transform: translateY(-1px);
}

.todo-form__button:active {
  transform: translateY(0);
}

.todo-app__main {
  background-color: #fff;
  box-shadow:
    0 4px 8px 0 rgba(0, 0, 0, 0.12),
    0 2px 4px 0 rgba(0, 0, 0, 0.08);
  border-top: 4px solid #6574cd;
  border-radius: 0.5rem;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.todo-list {
  max-height: 300px;
  overflow-y: auto;
  padding: 0;
  margin: 0;
  list-style: none;
}

.todo-list__item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 0;
  border-bottom: 1px solid #e2e8f0;
  transition: background-color 0.2s ease;
}

.todo-list__item:hover {
  background-color: #f7fafc;
}

.todo-list__item:last-child {
  border-bottom: none;
}

.todo-list__item-text {
  flex-grow: 1;
  margin: 0 1rem;
  color: #2d3748;
  word-break: break-word;
}

.todo-list__item--completed .todo-list__item-text {
  text-decoration: line-through;
  color: #a0aec0;
}

.btn {
  background: none;
  border: none;
  padding: 0.5rem;
  border-radius: 0.25rem;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

.btn--delete:hover {
  background-color: #fed7d7;
}

.btn--delete:hover path {
  fill: #e53e3e;
}

.btn--check:hover {
  background-color: #c6f6d5;
}

.btn--check path {
  fill: #a0aec0;
}

.todo-list__item--completed .btn--check path,
.btn--check:hover path {
  fill: #38a169;
}

.todo-list__empty {
  text-align: center;
  color: #a0aec0;
  padding: 2rem;
  font-style: italic;
}

.todo-app__footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 1.5rem;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.todo-app__footer-text {
  flex-grow: 1;
  color: #4a5568;
  font-weight: 500;
  margin: 0;
}

.btns {
  display: flex;
  justify-content: space-around;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.filter-btn {
  flex-grow: 1;
  padding: 0.5rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.25rem;
  background-color: #fff;
  color: #4a5568;
  font-size: 0.875rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.filter-btn:hover {
  background-color: #f7fafc;
  border-color: #cbd5e0;
}

.btn--active {
  background-color: #6574cd !important;
  color: white !important;
  border-color: #6574cd !important;
}

.btn--active:hover {
  background-color: #5661b3 !important;
  border-color: #5661b3 !important;
}

/* Добавляем стили для disabled кнопок */
.filter-btn:disabled,
.btn--delete:disabled,
.btn--clear:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.filter-btn:disabled:hover,
.btn--delete:disabled:hover,
.btn--clear:disabled:hover {
  background-color: #fff;
  border-color: #e2e8f0;
  transform: none;
}

.btn--clear:disabled {
  background-color: #e2e8f0;
  color: #a0aec0;
}

.btn--clear:disabled:hover {
  background-color: #e2e8f0;
}

.btn--clear {
  min-width: 220px;
  background-color: #e2e8f0;
  color: #4a5568;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 500;
}

.btn--clear:hover {
  background-color: #c6cace;
}

/* Адаптивность для мобильных устройств */
@media (max-width: 640px) {
  .container {
    margin: 10px auto;
    padding: 0 10px;
  }

  .todo-form {
    flex-direction: column;
    gap: 0;
  }

  .todo-form__input {
    padding: 1rem;
  }

  .todo-form__button {
    padding: 1rem;
    width: 100%;
  }

  .todo-form__button:disabled {
    background-color: #cbd5e0;
    cursor: not-allowed;
    transform: none;
  }

  .todo-app__footer {
    flex-direction: column;
    align-items: stretch;
    gap: 0.5rem;
  }

  .todo-app__footer-text {
    text-align: center;
    margin-bottom: 0.5rem;
  }
}
</style>

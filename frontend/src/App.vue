<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const api = axios.create({
  baseURL: 'https://fictional-rotary-phone-69xq6grgvw4j2xv9q-8000.app.github.dev/api',
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json'
  }
})

const tasks = ref([])
const title = ref('')
const description = ref('')
const loading = ref(false)
const error = ref('')

async function loadTasks() {
  loading.value = true
  error.value = ''
  try {
    const response = await api.get('/tasks')
    tasks.value = response.data
  } catch {
    error.value = 'Не удалось загрузить задачи.'
  } finally {
    loading.value = false
  }
}

async function addTask() {
  if (!title.value.trim()) return
  try {
    await api.post('/tasks', {
      title: title.value,
      description: description.value || null
    })
    title.value = ''
    description.value = ''
    await loadTasks()
  } catch {
    error.value = 'Не удалось создать задачу.'
  }
}

async function toggleTask(task) {
  try {
    await api.patch(`/tasks/${task.id}`, {
      completed: !task.completed
    })
    await loadTasks()
  } catch {
    error.value = 'Не удалось обновить задачу.'
  }
}

async function deleteTask(id) {
  try {
    await api.delete(`/tasks/${id}`)
    tasks.value = tasks.value.filter(task => task.id !== id)
  } catch {
    error.value = 'Не удалось удалить задачу.'
  }
}

onMounted(loadTasks)
</script>

<template>
  <main class="container">
    <h1>Менеджер задач</h1>

    <form @submit.prevent="addTask">
      <input v-model="title" placeholder="Название задачи" required />
      <textarea v-model="description" placeholder="Описание задачи"></textarea>
      <button type="submit">Добавить задачу</button>
    </form>

    <p v-if="loading">Загрузка...</p>
    <p v-if="error" class="error">{{ error }}</p>

    <ul>
      <li v-for="task in tasks" :key="task.id">
        <label>
          <input type="checkbox" :checked="task.completed" @change="toggleTask(task)" />
          <span :class="{ done: task.completed }">{{ task.title }}</span>
        </label>
        <p v-if="task.description">{{ task.description }}</p>
        <button @click="deleteTask(task.id)">Удалить</button>
      </li>
    </ul>
  </main>
</template>

<style>
.container { max-width: 700px; margin: 40px auto; padding: 20px; font-family: Arial, sans-serif; }
form { display: grid; gap: 10px; margin-bottom: 24px; }
input:not([type="checkbox"]), textarea { padding: 10px; }
button { padding: 8px 12px; cursor: pointer; }
li { margin-bottom: 16px; }
.done { text-decoration: line-through; color: gray; }
.error { color: #c00; }
</style>

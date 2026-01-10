<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const tasks = ref([])
const newTaskTitle = ref('')
const newTaskDescription = ref('')
const editingTask = ref(null)
const isLoading = ref(true)

// Use relative path for production (nginx proxy), absolute for local dev
const API_URL = import.meta.env.DEV ? 'http://localhost:8080/todos' : '/todos'

const handleMouseMove = (e, id) => {
  const card = document.getElementById(`card-${id}`)
  if (!card) return
  const rect = card.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  const centerX = rect.width / 2
  const centerY = rect.height / 2
  const rotateX = ((y - centerY) / centerY) * -10
  const rotateY = ((x - centerX) / centerX) * 10
  card.style.setProperty('--rx', `${rotateX}deg`)
  card.style.setProperty('--ry', `${rotateY}deg`)
}

const resetTilt = (id) => {
  const card = document.getElementById(`card-${id}`)
  if (card) {
    card.style.setProperty('--rx', '0deg')
    card.style.setProperty('--ry', '0deg')
  }
}

const fetchTasks = async () => {
  isLoading.value = true
  try {
    await new Promise((r) => setTimeout(r, 1200))
    const response = await axios.get(API_URL)
    tasks.value = response.data
  } catch (error) {
    console.error('Error fetching tasks:', error)
  } finally {
    isLoading.value = false
  }
}

const addTask = async () => {
  if (!newTaskTitle.value.trim()) return
  const newTask = {
    title: newTaskTitle.value,
    description: newTaskDescription.value,
    completed: false,
  }
  try {
    const response = await axios.post(API_URL, newTask)
    tasks.value.push(response.data)
    newTaskTitle.value = ''
    newTaskDescription.value = ''
  } catch (error) {
    console.error('Error adding task:', error)
  }
}

const deleteTask = async (id) => {
  try {
    await axios.delete(`${API_URL}/${id}`)
    tasks.value = tasks.value.filter((t) => t.id !== id)
  } catch (error) {
    console.error('Error deleting task:', error)
  }
}

const toggleCompletion = async (task) => {
  const updatedTask = { ...task, completed: !task.completed }
  try {
    await axios.put(`${API_URL}/${task.id}`, updatedTask)
    task.completed = updatedTask.completed
  } catch (error) {
    console.error('Error updating task completion:', error)
  }
}

const startEditing = (task) => {
  editingTask.value = { ...task }
}

const cancelEditing = () => {
  editingTask.value = null
}

const updateTask = async () => {
  if (!editingTask.value) return
  try {
    const response = await axios.put(`${API_URL}/${editingTask.value.id}`, editingTask.value)
    const index = tasks.value.findIndex((t) => t.id === editingTask.value.id)
    if (index !== -1) tasks.value[index] = response.data
    editingTask.value = null
  } catch (error) {
    console.error('Error updating task:', error)
  }
}

onMounted(fetchTasks)
</script>

<template>
  <div class="cosmic-canvas">
    <div class="starfield layer"></div>
    <div class="nebula layer"></div>
    <div class="scanlines layer"></div>
    <div class="aurora layer"></div>
  </div>

  <div class="hud">
    <header class="hero">
      <div class="identity">
        <div class="badge">QUANTUM OPS</div>
        <h1 class="logo">
          <span class="ring"></span>
          <span class="text">ASTRA // TASK CORE</span>
        </h1>
        <p class="subtitle">FUTURISTIC COMMAND SURFACE / EXPERIMENTAL BUILD</p>
      </div>
      <div class="system-panel">
        <div class="chip ok">SYSTEM ONLINE</div>
        <div class="chip pulse">LIVE SYNC</div>
        <div class="chip">{{ tasks.length }} ACTIVE</div>
      </div>
    </header>

    <section class="input-deck glass">
      <div class="input-wrap">
        <label>MISSION TITLE</label>
        <input
          v-model="newTaskTitle"
          placeholder="Ex: Decode hyperspace channel"
          @keyup.enter="addTask"
        />
        <span class="underline"></span>
      </div>
      <div class="input-wrap">
        <label>MISSION NOTES</label>
        <input
          v-model="newTaskDescription"
          placeholder="Attach intel, coordinates, access codes"
          @keyup.enter="addTask"
        />
        <span class="underline"></span>
      </div>
      <button class="cta" @click="addTask">
        <span class="halo"></span>
        <span class="cta-text">LAUNCH DIRECTIVE</span>
      </button>
    </section>

    <div v-if="isLoading" class="loader-shell">
      <div class="triple-ring">
        <span></span><span></span><span></span>
      </div>
      <p class="loading-copy">Spooling astral buffers...</p>
    </div>

    <div v-else class="task-space">
      <div v-if="tasks.length === 0" class="empty">
        <div class="void">∅</div>
        <p>Awaiting your next cosmic directive.</p>
      </div>

      <TransitionGroup name="stagger" tag="div" class="card-grid">
        <div
          v-for="task in tasks"
          :key="task.id"
          :id="`card-${task.id}`"
          class="task-card"
          :class="{ done: task.completed }"
          @mousemove="handleMouseMove($event, task.id)"
          @mouseleave="resetTilt(task.id)"
        >
          <div class="card-holo"></div>
          <div class="card-main">
            <div class="card-head">
              <button class="orb-toggle" @click="toggleCompletion(task)">
                <span class="orb" :class="{ active: task.completed }"></span>
              </button>
              <div class="title-block">
                <p class="eyebrow">MISSION</p>
                <h3>{{ task.title }}</h3>
              </div>
              <div class="id-chip">ID {{ task.id.toString().padStart(4, '0') }}</div>
            </div>

            <div class="card-body">
              <p class="copy" v-if="!(editingTask && editingTask.id === task.id)">{{ task.description || 'No intel provided.' }}</p>

              <div v-if="editingTask && editingTask.id === task.id" class="edit-zone">
                <input v-model="editingTask.title" class="edit-title" placeholder="Title" />
                <textarea v-model="editingTask.description" class="edit-notes" placeholder="Details"></textarea>
                <div class="edit-actions">
                  <button class="pill primary" @click="updateTask">Save</button>
                  <button class="pill ghost" @click="cancelEditing">Cancel</button>
                </div>
              </div>
            </div>

            <div class="card-foot" v-if="!(editingTask && editingTask.id === task.id)">
              <div class="actions">
                <span class="action" @click="startEditing(task)">Modify</span>
                <span class="action danger" @click="deleteTask(task.id)">Delete</span>
              </div>
              <div class="progress">
                <span :style="{ width: task.completed ? '100%' : '42%' }"></span>
              </div>
            </div>
          </div>
        </div>
      </TransitionGroup>
    </div>
  </div>
</template>

<style scoped>
@keyframes shimmer {
  0% { background-position: 0 50%; }
  100% { background-position: 200% 50%; }
}

@keyframes spin { to { transform: rotate(360deg); } }

.cosmic-canvas {
  position: fixed;
  inset: 0;
  overflow: hidden;
  z-index: -1;
}

.layer { position: absolute; inset: 0; }

.starfield {
  background: radial-gradient(circle at 20% 20%, rgba(0, 255, 255, 0.15), transparent 35%),
    radial-gradient(circle at 80% 30%, rgba(255, 0, 128, 0.2), transparent 35%),
    radial-gradient(circle at 50% 80%, rgba(0, 255, 140, 0.14), transparent 30%),
    #03030a;
  filter: saturate(1.2);
}

.nebula {
  background: repeating-linear-gradient(90deg, rgba(255,255,255,0.02) 0 2px, transparent 2px 8px);
  mix-blend-mode: screen;
  animation: shimmer 14s linear infinite;
}

.scanlines {
  background: repeating-linear-gradient(0deg, rgba(255,255,255,0.04) 0 1px, transparent 1px 3px);
  opacity: 0.25;
}

.aurora {
  background: radial-gradient(ellipse at 30% 10%, rgba(0, 255, 255, 0.35), transparent 40%),
    radial-gradient(ellipse at 80% 70%, rgba(255, 0, 128, 0.35), transparent 35%);
  filter: blur(60px);
  opacity: 0.7;
}

.hud {
  max-width: 1100px;
  margin: 0 auto;
  padding: 48px 20px 120px;
  color: #e8f6ff;
  position: relative;
}

.hero {
  display: flex;
  justify-content: space-between;
  gap: 20px;
  align-items: center;
  margin-bottom: 36px;
}

.identity .badge {
  display: inline-block;
  padding: 6px 12px;
  border: 1px solid rgba(255, 255, 255, 0.25);
  background: rgba(0, 0, 0, 0.35);
  letter-spacing: 3px;
  font-size: 12px;
  border-radius: 999px;
}

.logo {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 8px 0;
  font-family: 'Orbitron', sans-serif;
  letter-spacing: 2px;
}

.logo .ring {
  width: 42px; height: 42px;
  border-radius: 50%;
  border: 2px solid rgba(0, 255, 255, 0.4);
  box-shadow: 0 0 14px rgba(0, 255, 255, 0.5), 0 0 30px rgba(255, 0, 128, 0.35);
  position: relative;
}

.logo .ring::after {
  content: '';
  position: absolute;
  inset: 10px;
  border-radius: 50%;
  border: 2px solid rgba(255, 0, 128, 0.5);
  filter: blur(1px);
}

.logo .text { font-size: 1.6rem; font-weight: 800; }

.subtitle {
  margin: 0;
  color: rgba(232, 246, 255, 0.7);
  letter-spacing: 2px;
  font-size: 0.85rem;
}

.system-panel { display: flex; gap: 10px; flex-wrap: wrap; justify-content: flex-end; }

.chip {
  padding: 8px 12px;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  background: rgba(0, 0, 0, 0.4);
  font-size: 0.85rem;
  letter-spacing: 1px;
}

.chip.ok { color: #7efaff; border-color: rgba(126, 250, 255, 0.35); }
.chip.pulse { color: #ff9bf2; border-color: rgba(255, 155, 242, 0.3); box-shadow: 0 0 18px rgba(255, 155, 242, 0.3); }

.glass {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(18px);
  box-shadow: 0 18px 50px rgba(0, 0, 0, 0.45), inset 0 0 0 1px rgba(255, 255, 255, 0.05);
  border-radius: 18px;
}

.input-deck {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 18px;
  padding: 20px;
  align-items: end;
  margin-bottom: 32px;
}

.input-wrap { position: relative; display: flex; flex-direction: column; gap: 6px; }
.input-wrap label { font-size: 0.8rem; letter-spacing: 2px; color: rgba(232, 246, 255, 0.7); }
.input-wrap input {
  background: rgba(0, 0, 0, 0.35);
  border: 1px solid rgba(255, 255, 255, 0.15);
  padding: 14px 16px;
  color: #fff;
  border-radius: 12px;
  font-size: 0.95rem;
  outline: none;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.input-wrap input:focus {
  border-color: rgba(126, 250, 255, 0.6);
  box-shadow: 0 0 0 1px rgba(126, 250, 255, 0.3), 0 10px 28px rgba(0, 255, 255, 0.15);
}

.underline {
  position: absolute;
  left: 0; right: 0; bottom: 6px;
  height: 2px;
  background: linear-gradient(90deg, #7efaff, #ff9bf2, #7efaff);
  background-size: 200% 100%;
  opacity: 0;
  transition: opacity 0.2s ease;
  animation: shimmer 3s linear infinite;
}

.input-wrap input:focus + .underline { opacity: 1; }

.cta {
  align-self: stretch;
  border: none;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  color: #01111b;
  font-weight: 800;
  letter-spacing: 1px;
  padding: 16px 18px;
  border-radius: 14px;
  background: linear-gradient(120deg, #7efaff, #ff9bf2, #8c7bff);
  background-size: 200% 100%;
  animation: shimmer 6s linear infinite;
  box-shadow: 0 14px 40px rgba(126, 250, 255, 0.25);
}

.cta .halo {
  position: absolute;
  inset: -30%;
  background: radial-gradient(circle at 50% 50%, rgba(255,255,255,0.35), transparent 60%);
  filter: blur(22px);
  opacity: 0;
  transition: opacity 0.25s ease;
}

.cta:hover .halo { opacity: 1; }

.cta-text { position: relative; z-index: 1; }

.loader-shell { text-align: center; margin-top: 70px; color: rgba(232, 246, 255, 0.8); }

.triple-ring { display: inline-grid; place-items: center; position: relative; width: 120px; height: 120px; }
.triple-ring span {
  position: absolute;
  border: 2px solid rgba(126, 250, 255, 0.35);
  border-radius: 50%;
  width: 100%; height: 100%;
  animation: spin 6s linear infinite;
}

.triple-ring span:nth-child(2) { width: 70%; height: 70%; border-color: rgba(255, 155, 242, 0.4); animation-duration: 4s; animation-direction: reverse; }
.triple-ring span:nth-child(3) { width: 40%; height: 40%; border-color: rgba(140, 123, 255, 0.5); animation-duration: 2.6s; }

.loading-copy { margin-top: 14px; letter-spacing: 2px; font-size: 0.85rem; }

.task-space { margin-top: 10px; }
.empty { text-align: center; padding: 70px 0; color: rgba(232, 246, 255, 0.6); }
.empty .void { font-size: 3rem; color: #ff9bf2; margin-bottom: 10px; }

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 18px;
}

.task-card {
  position: relative;
  transform-style: preserve-3d;
  transform: perspective(900px) rotateX(var(--rx, 0deg)) rotateY(var(--ry, 0deg));
  transition: transform 0.15s ease, filter 0.2s ease;
}

.task-card:hover { filter: brightness(1.08); }

.task-card .card-holo {
  position: absolute;
  inset: 0;
  background: linear-gradient(135deg, rgba(126, 250, 255, 0.12), rgba(255, 155, 242, 0.12));
  border-radius: 16px;
  filter: blur(20px);
  opacity: 0.6;
  z-index: 0;
}

.card-main {
  position: relative;
  z-index: 1;
  padding: 18px;
  border-radius: 16px;
  background: rgba(0, 0, 0, 0.55);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(18px);
  box-shadow: 0 20px 45px rgba(0, 0, 0, 0.45);
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.task-card.done .card-main { border-color: rgba(126, 255, 172, 0.35); box-shadow: 0 18px 36px rgba(126, 255, 172, 0.2); }

.card-head {
  display: flex;
  align-items: center;
  gap: 12px;
}

.orb-toggle { background: transparent; border: none; padding: 0; cursor: pointer; }
.orb {
  width: 26px; height: 26px;
  border-radius: 50%;
  border: 2px solid rgba(126, 250, 255, 0.6);
  box-shadow: 0 0 12px rgba(126, 250, 255, 0.5);
  position: relative;
  transition: all 0.2s ease;
}
.orb::after {
  content: '';
  position: absolute;
  inset: 6px;
  border-radius: 50%;
  background: rgba(255, 155, 242, 0.7);
  opacity: 0;
  transition: opacity 0.2s ease;
}
.orb.active { border-color: #7efaac; box-shadow: 0 0 14px rgba(126, 255, 172, 0.7); }
.orb.active::after { opacity: 1; background: rgba(126, 255, 172, 0.9); }

.title-block { flex: 1; }
.eyebrow { margin: 0; font-size: 0.7rem; letter-spacing: 2px; color: rgba(232, 246, 255, 0.65); }
.title-block h3 { margin: 2px 0 0; font-size: 1.15rem; letter-spacing: 1px; }

.id-chip {
  padding: 6px 10px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.08);
  font-size: 0.8rem;
  letter-spacing: 1px;
}

.card-body .copy { margin: 0; color: rgba(232, 246, 255, 0.78); line-height: 1.6; }

.edit-zone { display: flex; flex-direction: column; gap: 10px; }
.edit-title, .edit-notes {
  width: 100%;
  background: rgba(0, 0, 0, 0.4);
  border: 1px solid rgba(255, 255, 255, 0.14);
  color: #fff;
  padding: 12px;
  border-radius: 12px;
  outline: none;
  font-family: 'Exo 2', sans-serif;
}
.edit-notes { min-height: 80px; resize: vertical; }

.edit-actions { display: flex; gap: 10px; justify-content: flex-end; }
.pill {
  border: none;
  padding: 10px 14px;
  border-radius: 999px;
  cursor: pointer;
  font-weight: 700;
  letter-spacing: 1px;
}
.pill.primary { background: linear-gradient(120deg, #7efaff, #8c7bff); color: #031019; }
.pill.ghost { background: rgba(255, 255, 255, 0.08); color: #fff; border: 1px solid rgba(255, 255, 255, 0.15); }

.card-foot { display: flex; align-items: center; gap: 14px; justify-content: space-between; }
.actions { display: flex; gap: 12px; letter-spacing: 1px; }
.action { cursor: pointer; color: #7efaff; font-weight: 700; }
.action.danger { color: #ff9bf2; }
.action:hover { text-decoration: underline; }

.progress {
  flex: 1;
  height: 6px;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 999px;
  overflow: hidden;
}
.progress span {
  display: block;
  height: 100%;
  background: linear-gradient(90deg, #7efaff, #ff9bf2);
  border-radius: inherit;
  box-shadow: 0 0 12px rgba(126, 250, 255, 0.4);
  transition: width 0.3s ease;
}

.stagger-enter-active, .stagger-leave-active { transition: all 0.35s ease; }
.stagger-enter-from { opacity: 0; transform: translateY(12px); }
.stagger-leave-to { opacity: 0; transform: translateY(-12px); }

@media (max-width: 780px) {
  .hero { flex-direction: column; align-items: flex-start; }
  .system-panel { justify-content: flex-start; }
}

@media (max-width: 520px) {
  .hud { padding: 36px 16px 90px; }
}
</style>

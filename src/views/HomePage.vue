<template>
  <ion-page>

    <!-- HEADER -->
    <ion-header class="ion-no-border custom-header">
      <ion-toolbar>
        <ion-title></ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">

      <div class="page-container">

        <!-- GREETING -->
        <div class="greeting">
          <h1>Hello, User!</h1>
          <p>Have a nice day.</p>
        </div>

        <!-- FILTER PILLS -->
        <div class="filter-pills">
          <button
            v-for="f in filters"
            :key="f.value"
            class="pill"
            :class="{ active: activeFilter === f.value }"
            @click="activeFilter = f.value"
          >
            {{ f.label }}
          </button>
        </div>

        <!-- TASK SUMMARY CARDS -->
        <div class="project-scroll">
          <div class="project-card" v-for="stat in taskStats" :key="stat.label">
            <div class="project-icon">
              <ion-icon :icon="stat.icon" />
            </div>
            <span class="project-label">{{ stat.label }}</span>
            <h3 class="project-title">{{ stat.value }}</h3>
            <span class="project-date">{{ stat.subtitle }}</span>
          </div>
        </div>

        <!-- PROGRESS -->
        <div class="section-header">
          <h2>Progress</h2>
          <a class="view-all" @click="openViewAll">
            View All <ion-icon :icon="arrowForward" />
          </a>
        </div>

        <!-- PROGRESS TASK LIST -->
        <div class="task-rows">
          <TaskCard
            v-for="task in filteredTasks"
            :key="task.id"
            :task="task"
            :menu-open="openMenuId === task.id"
            @toggle-menu="toggleMenu(task.id)"
            @edit="handleEdit(task)"
            @delete="handleDelete(task.id)"
            @toggle-status="handleToggle(task.id)"
          />

          <div v-if="filteredTasks.length === 0" class="empty-state">
            <div class="empty-icon">✓</div>
            <h2>No Tasks Found</h2>
            <p>You don't have any tasks yet.</p>
          </div>
        </div>

      </div>

      <!-- FAB -->
      <ion-fab vertical="bottom" horizontal="end" slot="fixed" class="add-fab">
        <ion-fab-button @click="showAddModal = true">
          <ion-icon :icon="addOutline" />
        </ion-fab-button>
      </ion-fab>

      <!-- ADD MODAL -->
      <ion-modal :is-open="showAddModal" @didDismiss="showAddModal = false">
        <ion-header class="custom-modal-header">
          <ion-toolbar>
            <ion-buttons slot="start">
              <ion-button class="close-btn" @click="showAddModal = false">
                <ion-icon :icon="arrowBackOutline" />
              </ion-button>
            </ion-buttons>
            <ion-title>Create a Task</ion-title>
          </ion-toolbar>
        </ion-header>

        <ion-content class="modal-content">
          <TaskForm
            @add-task="handleAddTask"
            @validation-error="handleValidationError"
          />
        </ion-content>
      </ion-modal>

      <!-- VIEW ALL MODAL -->
      <ion-modal :is-open="showViewAll" @didDismiss="showViewAll = false">
        <ion-header class="custom-modal-header">
          <ion-toolbar>
            <ion-buttons slot="start">
              <ion-button class="close-btn" @click="showViewAll = false">
                <ion-icon :icon="arrowBackOutline" />
              </ion-button>
            </ion-buttons>
            <ion-title>All Tasks</ion-title>
          </ion-toolbar>
        </ion-header>

        <ion-content class="modal-content">
          <div class="view-all-content">

            <div class="view-all-summary">
              <span>
                <strong>{{ tasks.length }}</strong>
                {{ tasks.length === 1 ? 'task' : 'tasks' }} total
              </span>
            </div>

            <div class="task-rows">
              <TaskCard
                v-for="task in tasks"
                :key="task.id"
                :task="task"
                :menu-open="openMenuId === task.id"
                @toggle-menu="toggleMenu(task.id)"
                @edit="handleEdit(task)"
                @delete="handleDelete(task.id)"
                @toggle-status="handleToggle(task.id)"
              />

              <div v-if="tasks.length === 0" class="empty-state">
                <div class="empty-icon">✓</div>
                <h2>No Tasks Yet</h2>
                <p>Tap the + button to create your first task.</p>
              </div>
            </div>

          </div>
        </ion-content>
      </ion-modal>

      <!-- EDIT MODAL -->
      <ion-modal :is-open="showEditModal" @didDismiss="closeEditModal">
        <ion-header class="custom-modal-header">
          <ion-toolbar>
            <ion-buttons slot="start">
              <ion-button class="close-btn" @click="closeEditModal">
                <ion-icon :icon="arrowBackOutline" />
              </ion-button>
            </ion-buttons>
            <ion-title>Edit Task</ion-title>
          </ion-toolbar>
        </ion-header>

        <ion-content class="modal-content">
          <div class="edit-form">

            <ion-item class="custom-input-item" lines="none">
              <ion-input v-model="editingTask.title" label="Task Title" label-placement="stacked" placeholder="Enter task title" />
            </ion-item>

            <ion-item class="custom-input-item" lines="none">
              <ion-textarea v-model="editingTask.description" label="Description" label-placement="stacked" placeholder="Enter task description" :auto-grow="true" />
            </ion-item>

            <div class="custom-input-item date-item">
              <div class="date-icon-wrap">
                <ion-icon :icon="calendarOutline" />
              </div>
              <ion-input
                v-model="editingTask.dueDate"
                type="date"
                label="Due Date"
                label-placement="stacked"
                :min="localToday"
                class="date-input"
              />
            </div>

            <p class="date-note">Past dates cannot be selected.</p>

            <ion-item class="custom-input-item" lines="none">
              <ion-select v-model="editingTask.priority" label="Priority" label-placement="stacked" interface="popover">
                <ion-select-option value="Low">Low</ion-select-option>
                <ion-select-option value="Medium">Medium</ion-select-option>
                <ion-select-option value="High">High</ion-select-option>
              </ion-select>
            </ion-item>

            <ion-item class="custom-input-item" lines="none">
              <ion-select v-model="editingTask.status" label="Status" label-placement="stacked" interface="popover">
                <ion-select-option value="Pending">Pending</ion-select-option>
                <ion-select-option value="Completed">Completed</ion-select-option>
              </ion-select>
            </ion-item>

            <ion-button expand="block" class="save-button" @click="saveEdit">Save Changes</ion-button>
            <ion-button expand="block" fill="outline" class="cancel-button" @click="closeEditModal">Cancel</ion-button>
          </div>
        </ion-content>
      </ion-modal>

      <!-- CUSTOM CONFIRM DIALOG -->
      <transition name="dialog-fade">
        <div v-if="confirmState.visible" class="dialog-overlay" @click.self="closeConfirm(false)">
          <div class="dialog-box">
            <div class="dialog-icon" :class="confirmState.variant">
              <ion-icon :icon="confirmState.variant === 'danger' ? trashOutline : informationCircleOutline" />
            </div>

            <h3 class="dialog-title">{{ confirmState.title }}</h3>
            <p class="dialog-message">{{ confirmState.message }}</p>

            <div class="dialog-actions">
              <button class="dialog-btn dialog-btn-cancel" @click="closeConfirm(false)">
                {{ confirmState.cancelText || 'Cancel' }}
              </button>
              <button
                class="dialog-btn dialog-btn-confirm"
                :class="{ 'is-danger': confirmState.variant === 'danger' }"
                @click="closeConfirm(true)"
              >
                {{ confirmState.confirmText || 'Confirm' }}
              </button>
            </div>
          </div>
        </div>
      </transition>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonButtons,
  IonButton, IonIcon, IonModal, IonItem, IonInput, IonTextarea,
  IonSelect, IonSelectOption, IonFab, IonFabButton
} from '@ionic/vue'
import {
  addOutline, calendarOutline, arrowBackOutline, arrowForward,
  listOutline, timeOutline, checkmarkDoneOutline,
  trashOutline, informationCircleOutline
} from 'ionicons/icons'

import TaskForm from '@/components/TaskForm.vue'
import TaskCard from '@/components/TaskCard.vue'

import { ref as firebaseRef, push, set, update, remove, onValue } from 'firebase/database'
import { db } from '@/firebase'

/* =========================
   TASK INTERFACE
========================= */
interface Task {
  id: string
  title: string
  description: string
  dueDate: string
  priority: string
  status: string
}

/* =========================
   FILTERS
========================= */
const filters = [
  { label: 'My Tasks',    value: 'All' },
  { label: 'In Progress', value: 'Pending' },
  { label: 'Completed',   value: 'Completed' }
]
const activeFilter = ref('All')

/* =========================
   TASKS
========================= */
const tasks = ref<Task[]>([])

const date = new Date()
const localToday = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`

const filteredTasks = computed(() => {
  if (activeFilter.value === 'All') return tasks.value
  return tasks.value.filter(t => t.status === activeFilter.value)
})

/* =========================
   TASK SUMMARY STATS
========================= */
const taskStats = computed(() => {
  const total = tasks.value.length
  const pending = tasks.value.filter(t => t.status === 'Pending').length
  const completed = tasks.value.filter(t => t.status === 'Completed').length

  return [
    { label: 'Total Tasks', value: total,     subtitle: 'Tasks in list',  icon: listOutline },
    { label: 'Pending',     value: pending,   subtitle: 'Tasks to do',    icon: timeOutline },
    { label: 'Completed',   value: completed, subtitle: 'Tasks done',     icon: checkmarkDoneOutline }
  ]
})

/* =========================
   VIEW ALL MODAL
========================= */
const showViewAll = ref(false)
const openViewAll = () => {
  openMenuId.value = null
  showViewAll.value = true
}

/* =========================
   CUSTOM CONFIRM DIALOG
========================= */
interface ConfirmOptions {
  title: string
  message: string
  confirmText?: string
  cancelText?: string
  variant?: 'default' | 'danger'
}

const confirmState = reactive({
  visible: false,
  title: '',
  message: '',
  confirmText: 'Confirm',
  cancelText: 'Cancel',
  variant: 'default' as 'default' | 'danger',
  resolve: null as null | ((v: boolean) => void)
})

const showConfirm = (options: ConfirmOptions): Promise<boolean> => {
  confirmState.title = options.title
  confirmState.message = options.message
  confirmState.confirmText = options.confirmText || 'Confirm'
  confirmState.cancelText = options.cancelText || 'Cancel'
  confirmState.variant = options.variant || 'default'
  confirmState.visible = true

  return new Promise<boolean>((resolve) => {
    confirmState.resolve = resolve
  })
}

const closeConfirm = (result: boolean) => {
  confirmState.visible = false
  if (confirmState.resolve) {
    confirmState.resolve(result)
    confirmState.resolve = null
  }
}

/* =========================
   VALIDATION ERROR
========================= */
const handleValidationError = async (payload: { title: string; message: string }) => {
  await showConfirm({
    title: payload.title,
    message: payload.message,
    confirmText: 'OK'
  })
}

/* =========================
   FIREBASE LISTENER
========================= */
let unsubscribe: (() => void) | null = null

onMounted(() => {
  const tasksRef = firebaseRef(db, 'tasks')
  unsubscribe = onValue(tasksRef, (snapshot) => {
    const data = snapshot.val()
    if (!data) { tasks.value = []; return }

    const loaded: Task[] = Object.entries(data).map(([id, task]: [string, any]) => ({
      id,
      title: task.title || '',
      description: task.description || '',
      dueDate: task.dueDate || '',
      priority: task.priority || 'Medium',
      status: task.status || 'Pending'
    }))

    loaded.sort((a, b) => a.dueDate.localeCompare(b.dueDate))
    tasks.value = loaded
  })
})

onUnmounted(() => { if (unsubscribe) unsubscribe() })

/* =========================
   ADD TASK
========================= */
const showAddModal = ref(false)

const handleAddTask = async (newTask: Omit<Task, 'id'>) => {
  try {
    if (!newTask.title.trim()) {
      await showConfirm({ title: 'Missing Title', message: 'Please enter a task title before continuing.', confirmText: 'OK' })
      return
    }
    if (!newTask.dueDate) {
      await showConfirm({ title: 'Missing Due Date', message: 'Please select a due date for this task.', confirmText: 'OK' })
      return
    }
    if (newTask.dueDate < localToday) {
      await showConfirm({ title: 'Invalid Date', message: 'Please select today or a future date.', confirmText: 'OK' })
      return
    }

    const newRef = push(firebaseRef(db, 'tasks'))
    await set(newRef, {
      title: newTask.title,
      description: newTask.description,
      dueDate: newTask.dueDate,
      priority: newTask.priority,
      status: 'Pending',
      createdAt: Date.now(),
      updatedAt: Date.now()
    })

    showAddModal.value = false
  } catch (err) {
    console.error(err)
    await showConfirm({ title: 'Error', message: 'Failed to add task. Please try again.', confirmText: 'OK', variant: 'danger' })
  }
}

/* =========================
   DELETE / TOGGLE
========================= */
const handleDelete = async (id: string) => {
  const task = tasks.value.find(t => t.id === id)
  const taskName = task?.title || 'this task'

  const confirmed = await showConfirm({
    title: 'Delete Task',
    message: `Are you sure you want to delete "${taskName}"? This action cannot be undone.`,
    confirmText: 'Delete',
    cancelText: 'Cancel',
    variant: 'danger'
  })

  if (!confirmed) return

  try {
    await remove(firebaseRef(db, `tasks/${id}`))
    openMenuId.value = null
  } catch (err) {
    console.error(err)
    await showConfirm({ title: 'Error', message: 'Failed to delete task.', confirmText: 'OK', variant: 'danger' })
  }
}

const handleToggle = async (id: string) => {
  const task = tasks.value.find(t => t.id === id)
  if (!task) return
  const newStatus = task.status === 'Pending' ? 'Completed' : 'Pending'
  try {
    await update(firebaseRef(db, `tasks/${id}`), {
      status: newStatus,
      updatedAt: Date.now()
    })
    openMenuId.value = null
  } catch (err) {
    console.error(err)
  }
}

/* =========================
   EDIT MODAL
========================= */
const showEditModal = ref(false)
const editingTask = reactive<Task>({
  id: '', title: '', description: '', dueDate: '', priority: 'Medium', status: 'Pending'
})

const handleEdit = (task: Task) => {
  Object.assign(editingTask, task)
  showEditModal.value = true
  openMenuId.value = null
}

const saveEdit = async () => {
  if (!editingTask.title.trim()) {
    await showConfirm({ title: 'Missing Title', message: 'Please enter a task title.', confirmText: 'OK' })
    return
  }
  if (!editingTask.dueDate) {
    await showConfirm({ title: 'Missing Due Date', message: 'Please select a due date.', confirmText: 'OK' })
    return
  }
  if (editingTask.dueDate < localToday) {
    await showConfirm({ title: 'Invalid Date', message: 'Please select today or a future date.', confirmText: 'OK' })
    return
  }

  try {
    await update(firebaseRef(db, `tasks/${editingTask.id}`), {
      title: editingTask.title,
      description: editingTask.description,
      dueDate: editingTask.dueDate,
      priority: editingTask.priority,
      status: editingTask.status,
      updatedAt: Date.now()
    })
    showEditModal.value = false
  } catch (err) {
    console.error(err)
    await showConfirm({ title: 'Error', message: 'Failed to update task.', confirmText: 'OK', variant: 'danger' })
  }
}

const closeEditModal = () => { showEditModal.value = false }

/* =========================
   DROPDOWN MENU STATE
========================= */
const openMenuId = ref<string | null>(null)
const toggleMenu = (id: string) => {
  openMenuId.value = openMenuId.value === id ? null : id
}
</script>

<style scoped>
/* =========================================================
   SAGE GREEN THEME
========================================================= */
ion-content { --background: #f4f7f5; }

/* ================= HEADER ================= */
.custom-header ion-toolbar {
  --background: #f4f7f5;
  --color: #2d4538;
  --min-height: 0;
  --padding-top: 0;
  --padding-bottom: 0;

  /* ✅ More generous top padding so greeting has plenty of breathing room */
  padding-top: max(env(safe-area-inset-top), 48px);
}

ion-title { padding: 0; }

/* ================= PAGE ================= */
.page-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 24px 20px 120px;
}

/* ================= GREETING ================= */
.greeting { margin-bottom: 24px; animation: fadeSlideUp 0.4s ease both; }

.greeting h1 {
  margin: 0;
  font-size: 26px;
  font-weight: 800;
  color: #2d4538;
  letter-spacing: -0.02em;
}

.greeting p {
  margin: 4px 0 0;
  font-size: 14px;
  color: #557a66;
  font-weight: 500;
}

/* ================= FILTER PILLS ================= */
.filter-pills {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
  overflow-x: auto;
  padding-bottom: 4px;
  animation: fadeSlideUp 0.5s ease both;
}

.filter-pills::-webkit-scrollbar { display: none; }

.pill {
  flex-shrink: 0;
  padding: 9px 18px;
  border-radius: 999px;
  border: 1.5px solid #d3e0d7;
  background: #ffffff;
  color: #557a66;
  font-size: 0.82rem;
  font-weight: 700;
  cursor: pointer;
  font-family: inherit;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.pill:hover { border-color: #3b5e4c; color: #3b5e4c; }

.pill.active {
  background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  color: #ffffff;
  border-color: transparent;
  box-shadow: 0 8px 18px -8px rgba(59, 94, 76, 0.6);
}

/* ================= SUMMARY CARDS ================= */
.project-scroll {
  display: flex;
  gap: 14px;
  overflow-x: auto;
  padding: 4px 0 18px;
  margin: 0 -20px 12px;
  padding-left: 20px;
  padding-right: 20px;
  scroll-snap-type: x mandatory;
  animation: fadeSlideUp 0.6s ease both;
}

.project-scroll::-webkit-scrollbar { display: none; }

.project-card {
  flex: 0 0 180px;
  padding: 16px 18px;
  border-radius: 20px;
  background: linear-gradient(135deg, #3b5e4c 0%, #557a66 60%, #739882 100%);
  color: #ffffff;
  scroll-snap-align: start;
  position: relative;
  overflow: hidden;
  box-shadow:
    0 16px 32px -16px rgba(59, 94, 76, 0.6),
    inset 0 1px 0 rgba(255, 255, 255, 0.15);
  transition: transform 0.25s ease;
  cursor: pointer;
}

.project-card::before {
  content: "";
  position: absolute;
  top: -40%; right: -30%;
  width: 140px; height: 140px;
  background: radial-gradient(circle, rgba(255,255,255,0.18) 0%, transparent 70%);
  border-radius: 50%;
}

.project-card:hover { transform: translateY(-3px); }

.project-icon {
  width: 40px;
  height: 40px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.18);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 14px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  position: relative;
  z-index: 1;
}

.project-icon ion-icon { font-size: 22px; color: #ffffff; }

.project-label {
  font-size: 0.68rem;
  font-weight: 700;
  color: rgba(255, 255, 255, 0.75);
  text-transform: uppercase;
  letter-spacing: 0.08em;
  position: relative;
  z-index: 1;
}

.project-title {
  margin: 6px 0 14px;
  font-size: 1.75rem;
  font-weight: 800;
  color: #ffffff;
  line-height: 1.1;
  letter-spacing: -0.02em;
  position: relative;
  z-index: 1;
}

.project-date {
  font-size: 0.72rem;
  font-weight: 600;
  color: rgba(255, 255, 255, 0.7);
  position: relative;
  z-index: 1;
}

/* ================= SECTION HEADER ================= */
.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin: 8px 0 14px;
  animation: fadeSlideUp 0.7s ease both;
}

.section-header h2 {
  margin: 0;
  font-size: 1.2rem;
  font-weight: 800;
  color: #2d4538;
  letter-spacing: -0.01em;
}

.view-all {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  font-size: 0.82rem;
  font-weight: 700;
  color: #3b5e4c;
  cursor: pointer;
  transition: gap 0.2s ease;
}

.view-all:hover { gap: 8px; }
.view-all ion-icon { font-size: 16px; }

/* ================= TASK LIST CONTAINER ================= */
.task-rows {
  display: flex;
  flex-direction: column;
  gap: 12px;
  animation: fadeSlideUp 0.8s ease both;
}

/* ================= EMPTY STATE ================= */
.empty-state {
  text-align: center;
  padding: 50px 24px;
  background: #ffffff;
  border-radius: 20px;
  border: 1px dashed #c9d9ce;
}

.empty-icon {
  width: 64px; height: 64px;
  margin: 0 auto 16px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #e8f5e9, #d1e7d5);
  color: #3b5e4c;
  font-size: 28px;
  font-weight: 800;
}

.empty-state h2 {
  margin: 0 0 6px;
  font-size: 18px;
  font-weight: 800;
  color: #2d4538;
}

.empty-state p {
  margin: 0;
  font-size: 13px;
  color: #6b8275;
}

/* ================= FAB ================= */
.add-fab {
  margin-bottom: 24px;
  margin-right: 20px;
}

.add-fab ion-fab-button {
  --background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  --background-activated: #284033;
  --box-shadow: 0 12px 28px -6px rgba(59, 94, 76, 0.6);
  --border-radius: 18px;
  width: 58px;
  height: 58px;
}

.add-fab ion-icon { font-size: 28px; color: #ffffff; }

/* ================= MODALS ================= */
.custom-modal-header ion-toolbar {
  --background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  --color: #ffffff;
  padding-top: env(safe-area-inset-top, 4px);
}

.close-btn { --color: #ffffff; font-weight: 600; }

.modal-content {
  --background: #f4f7f5;
  --color: #1c2e24;
}

/* ================= VIEW ALL MODAL ================= */
.view-all-content {
  max-width: 1000px;
  margin: 0 auto;
  padding: 16px 20px 40px;
}

.view-all-summary {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  padding: 12px 16px;
  background: #ffffff;
  border: 1px solid #e1e9e3;
  border-radius: 14px;
  margin-bottom: 16px;
  font-size: 0.82rem;
  font-weight: 600;
  color: #557a66;
  box-shadow: 0 4px 12px rgba(45, 69, 56, 0.04);
}

.view-all-summary strong {
  font-size: 1rem;
  font-weight: 800;
  color: #3b5e4c;
  margin-right: 6px;
}

/* ================= EDIT FORM ================= */
.edit-form {
  max-width: 700px;
  margin: 0 auto;
  padding: 16px 20px 40px;
}

.custom-input-item {
  margin-bottom: 14px;
  --background: #ffffff;
  --border-radius: 12px;
  --highlight-color-focused: #3b5e4c;
  --color: #1c2e24;
  --min-height: 56px;
  --padding-start: 12px;
  --padding-end: 12px;
  --inner-padding-end: 0;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.03);
  border: 1px solid #e1e9e3;
  border-radius: 12px;
}

.custom-input-item ion-input,
.custom-input-item ion-textarea,
.custom-input-item ion-select {
  --color: #1c2e24 !important;
  --placeholder-color: #a8b8ae !important;
  --placeholder-opacity: 1 !important;
  color: #1c2e24 !important;
}

.custom-input-item ion-input::part(native),
.custom-input-item ion-textarea::part(native) {
  color: #1c2e24 !important;
  background: transparent !important;
}

.custom-input-item ion-select::part(text) {
  color: #1c2e24 !important;
  opacity: 1 !important;
}

.custom-input-item ion-select::part(placeholder) {
  color: #a8b8ae !important;
  opacity: 1 !important;
}

.custom-input-item ion-label,
:deep(.custom-input-item .label-text) {
  color: #3b5e4c !important;
  font-weight: 700 !important;
  font-size: 0.78rem !important;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

.date-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 6px 14px;
  background: #ffffff;
  border: 1px solid #e1e9e3;
  border-radius: 12px;
  margin-bottom: 6px;
  transition: all 0.2s ease;
}

.date-item:focus-within {
  border-color: #3b5e4c;
  box-shadow: 0 0 0 3px rgba(59, 94, 76, 0.15);
}

.date-icon-wrap {
  width: 36px;
  height: 36px;
  border-radius: 10px;
  background: linear-gradient(135deg, #3b5e4c, #557a66);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  box-shadow: 0 4px 10px -3px rgba(59, 94, 76, 0.5);
}

.date-icon-wrap ion-icon { font-size: 18px; color: #ffffff; }

.date-input {
  flex: 1;
  --padding-start: 0;
  --padding-end: 0;
  --color: #1c2e24 !important;
  color: #1c2e24 !important;
}

.date-input::part(native) {
  color: #1c2e24 !important;
  background: transparent !important;
}

.date-input::part(native)::-webkit-calendar-picker-indicator {
  opacity: 0;
  position: absolute;
  right: 0;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.date-item ion-label,
:deep(.date-item .label-text) {
  color: #3b5e4c !important;
  font-weight: 700 !important;
  font-size: 0.78rem !important;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

.date-note {
  margin: -6px 0 16px 12px;
  font-size: 12px;
  color: #6b8275;
}

.save-button {
  margin-top: 20px;
  margin-bottom: 12px;
  --background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  --background-hover: linear-gradient(135deg, #314e3f 0%, #4a6f5b 100%);
  --background-activated: #284033;
  --color: #ffffff;
  --border-radius: 12px;
  height: 48px;
  font-weight: 700;
}

.cancel-button {
  --color: #3b5e4c;
  --border-color: #3b5e4c;
  --border-width: 1.5px;
  --border-radius: 12px;
  height: 48px;
  font-weight: 600;
}

/* ================= CONFIRM DIALOG ================= */
.dialog-overlay {
  position: fixed;
  inset: 0;
  background: rgba(15, 30, 22, 0.55);
  backdrop-filter: blur(6px);
  -webkit-backdrop-filter: blur(6px);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  z-index: 99999;
}

.dialog-box {
  width: 100%;
  max-width: 340px;
  background: #ffffff;
  border-radius: 22px;
  padding: 26px 22px 20px;
  text-align: center;
  box-shadow:
    0 24px 60px -12px rgba(15, 30, 22, 0.4),
    0 8px 20px -6px rgba(45, 69, 56, 0.15);
  animation: dialogPop 0.22s cubic-bezier(0.34, 1.56, 0.64, 1) both;
  border: 1px solid #e1e9e3;
}

@keyframes dialogPop {
  from { opacity: 0; transform: scale(0.9) translateY(10px); }
  to   { opacity: 1; transform: scale(1) translateY(0); }
}

.dialog-icon {
  width: 60px;
  height: 60px;
  margin: 0 auto 16px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #eaf0ec 0%, #d1e7d5 100%);
  color: #3b5e4c;
  box-shadow: 0 8px 20px -8px rgba(59, 94, 76, 0.4);
}

.dialog-icon.danger {
  background: linear-gradient(135deg, #fee2e2 0%, #fecaca 100%);
  color: #dc2626;
  box-shadow: 0 8px 20px -8px rgba(220, 38, 38, 0.4);
}

.dialog-icon ion-icon { font-size: 28px; }

.dialog-title {
  margin: 0 0 8px;
  font-size: 1.15rem;
  font-weight: 800;
  color: #2d4538;
  letter-spacing: -0.01em;
}

.dialog-message {
  margin: 0 0 22px;
  font-size: 0.88rem;
  font-weight: 500;
  color: #557a66;
  line-height: 1.5;
  word-break: break-word;
}

.dialog-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.dialog-btn {
  padding: 12px 16px;
  border-radius: 12px;
  border: none;
  font-size: 0.88rem;
  font-weight: 800;
  cursor: pointer;
  font-family: inherit;
  letter-spacing: 0.02em;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}

.dialog-btn:active { transform: scale(0.97); }

.dialog-btn-cancel {
  background: #f4f7f5;
  color: #557a66;
  border: 1.5px solid #d3e0d7;
}

.dialog-btn-cancel:hover { background: #eaf0ec; color: #3b5e4c; }

.dialog-btn-confirm {
  background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  color: #ffffff;
  box-shadow: 0 8px 18px -8px rgba(59, 94, 76, 0.6);
}

.dialog-btn-confirm:hover {
  box-shadow: 0 10px 22px -8px rgba(59, 94, 76, 0.7);
}

.dialog-btn-confirm.is-danger {
  background: linear-gradient(135deg, #dc2626 0%, #b91c1c 100%);
  box-shadow: 0 8px 18px -8px rgba(220, 38, 38, 0.6);
}

.dialog-btn-confirm.is-danger:hover {
  box-shadow: 0 10px 22px -8px rgba(220, 38, 38, 0.7);
}

.dialog-fade-enter-active,
.dialog-fade-leave-active {
  transition: opacity 0.2s ease;
}

.dialog-fade-enter-from,
.dialog-fade-leave-to {
  opacity: 0;
}

/* ================= ANIMATION ================= */
@keyframes fadeSlideUp {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ================= MOBILE ================= */
@media (max-width: 600px) {
  /* ✅ Even more top space on mobile */
  .custom-header ion-toolbar {
    padding-top: max(env(safe-area-inset-top), 60px);
  }

  .page-container { padding: 28px 16px 120px; }
  .greeting h1 { font-size: 22px; }
  .project-scroll { margin: 0 -16px 12px; padding-left: 16px; padding-right: 16px; }
  .view-all-content { padding: 12px 16px 40px; }
}
</style>
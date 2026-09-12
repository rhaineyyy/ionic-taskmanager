<template>
  <ion-page>
    <!-- Header -->
    <ion-header>
      <ion-toolbar>
        <ion-title>Daily Task Manager</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div class="page-container">

        <!-- Page Header -->
        <div class="page-heading">
          <h1>My Daily Tasks</h1>
          <p>Manage your tasks and stay organized.</p>
        </div>

        <!-- Task Summary -->
        <TaskSummary :tasks="tasks" />

        <!-- Add Task Form -->
        <TaskForm @add-task="addTask" />

        <!-- Task List -->
        <TaskList
          :tasks="tasks"
          @edit-task="editTask"
          @delete-task="deleteTask"
          @toggle-status="toggleStatus"
        />

      </div>

      <!-- Edit Task Modal -->
      <ion-modal
        :is-open="showEditModal"
        @didDismiss="closeEditModal"
      >
        <ion-header>
          <ion-toolbar>
            <ion-title>Edit Task</ion-title>

            <ion-buttons slot="end">
              <ion-button @click="closeEditModal">
                Close
              </ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>

        <ion-content class="ion-padding">
          <div class="edit-form">

            <!-- Title -->
            <ion-item>
              <ion-input
                v-model="editingTask.title"
                label="Task Title"
                label-placement="stacked"
                placeholder="Enter task title"
              />
            </ion-item>

            <!-- Description -->
            <ion-item>
              <ion-textarea
                v-model="editingTask.description"
                label="Description"
                label-placement="stacked"
                placeholder="Enter task description"
                :auto-grow="true"
              />
            </ion-item>

            <!-- Due Date -->
            <ion-item>
              <ion-input
                v-model="editingTask.dueDate"
                type="date"
                label="Due Date"
                label-placement="stacked"
                :min="localToday"
              />
            </ion-item>

            <p class="date-note">
              Past dates cannot be selected.
            </p>

            <!-- Priority -->
            <ion-item>
              <ion-select
                v-model="editingTask.priority"
                label="Priority"
                label-placement="stacked"
                interface="popover"
              >
                <ion-select-option value="Low">
                  Low
                </ion-select-option>

                <ion-select-option value="Medium">
                  Medium
                </ion-select-option>

                <ion-select-option value="High">
                  High
                </ion-select-option>
              </ion-select>
            </ion-item>

            <!-- Status -->
            <ion-item>
              <ion-select
                v-model="editingTask.status"
                label="Status"
                label-placement="stacked"
                interface="popover"
              >
                <ion-select-option value="Pending">
                  Pending
                </ion-select-option>

                <ion-select-option value="Completed">
                  Completed
                </ion-select-option>
              </ion-select>
            </ion-item>

            <!-- Save Button -->
            <ion-button
              expand="block"
              class="save-button"
              @click="saveEdit"
            >
              Save Changes
            </ion-button>

            <!-- Cancel Button -->
            <ion-button
              expand="block"
              fill="outline"
              @click="closeEditModal"
            >
              Cancel
            </ion-button>

          </div>
        </ion-content>
      </ion-modal>

    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'

import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonModal,
  IonButtons,
  IonButton,
  IonItem,
  IonInput,
  IonTextarea,
  IonSelect,
  IonSelectOption
} from '@ionic/vue'

import TaskForm from '@/components/TaskForm.vue'
import TaskList from '@/components/TaskList.vue'
import TaskSummary from '@/components/TaskSummary.vue'


/* =========================
   TASK INTERFACE
========================= */

interface Task {
  id: number
  title: string
  description: string
  dueDate: string
  priority: string
  status: string
}


/* =========================
   TASK DATA
========================= */

const tasks = ref<Task[]>([])


/* =========================
   TODAY'S DATE
   Used to disable past dates
========================= */

const date = new Date()

const localToday = `${date.getFullYear()}-${String(
  date.getMonth() + 1
).padStart(2, '0')}-${String(
  date.getDate()
).padStart(2, '0')}`


/* =========================
   ADD TASK
========================= */

const addTask = (newTask: Omit<Task, 'id'>) => {
  const task: Task = {
    id: Date.now(),
    title: newTask.title,
    description: newTask.description,
    dueDate: newTask.dueDate,
    priority: newTask.priority,
    status: newTask.status
  }

  tasks.value.push(task)
}


/* =========================
   DELETE TASK
========================= */

const deleteTask = (id: number) => {
  const confirmed = window.confirm(
    'Are you sure you want to delete this task?'
  )

  if (!confirmed) {
    return
  }

  tasks.value = tasks.value.filter(
    task => task.id !== id
  )
}


/* =========================
   TOGGLE TASK STATUS
========================= */

const toggleStatus = (id: number) => {
  const task = tasks.value.find(
    task => task.id === id
  )

  if (!task) {
    return
  }

  task.status =
    task.status === 'Pending'
      ? 'Completed'
      : 'Pending'
}


/* =========================
   EDIT MODAL
========================= */

const showEditModal = ref(false)

const editingTask = reactive<Task>({
  id: 0,
  title: '',
  description: '',
  dueDate: '',
  priority: 'Medium',
  status: 'Pending'
})


/* =========================
   OPEN EDIT MODAL
========================= */

const editTask = (task: Task) => {
  editingTask.id = task.id
  editingTask.title = task.title
  editingTask.description = task.description
  editingTask.dueDate = task.dueDate
  editingTask.priority = task.priority
  editingTask.status = task.status

  showEditModal.value = true
}


/* =========================
   SAVE EDIT
========================= */

const saveEdit = () => {

  if (!editingTask.title.trim()) {
    window.alert('Please enter a task title.')
    return
  }

  if (!editingTask.dueDate) {
    window.alert('Please select a due date.')
    return
  }

  // Prevent past dates even if someone manually enters one
  if (editingTask.dueDate < localToday) {
    window.alert('Please select today or a future date.')
    return
  }

  const task = tasks.value.find(
    task => task.id === editingTask.id
  )

  if (!task) {
    return
  }

  task.title = editingTask.title
  task.description = editingTask.description
  task.dueDate = editingTask.dueDate
  task.priority = editingTask.priority
  task.status = editingTask.status

  showEditModal.value = false
}


/* =========================
   CLOSE EDIT MODAL
========================= */

const closeEditModal = () => {
  showEditModal.value = false
}
</script>


<style scoped>
/* =========================
   PAGE
========================= */

ion-content {
  --background: #f5f7fa;
}

.page-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}


/* =========================
   HEADER
========================= */

ion-toolbar {
  --background: #ffffff;
  --color: #1f2937;
  border-bottom: 1px solid #e5e7eb;
}

ion-title {
  font-weight: 700;
}


/* =========================
   PAGE HEADING
========================= */

.page-heading {
  margin-bottom: 20px;
}

.page-heading h1 {
  margin: 0;
  font-size: 28px;
  font-weight: 700;
  color: #1f2937;
}

.page-heading p {
  margin-top: 6px;
  margin-bottom: 0;
  color: #6b7280;
  font-size: 15px;
}


/* =========================
   EDIT FORM
========================= */

.edit-form {
  max-width: 700px;
  margin: 0 auto;
}

.edit-form ion-item {
  margin-bottom: 12px;
  --border-radius: 10px;
  --background: #ffffff;
}


/* =========================
   DATE NOTE
========================= */

.date-note {
  margin: -4px 0 15px 16px;
  font-size: 13px;
  color: #6b7280;
}


/* =========================
   BUTTONS
========================= */

.save-button {
  margin-top: 25px;
  margin-bottom: 12px;
  --border-radius: 10px;
  height: 48px;
  font-weight: 600;
}


/* =========================
   MOBILE
========================= */

@media (max-width: 600px) {
  .page-container {
    padding: 15px;
  }

  .page-heading h1 {
    font-size: 24px;
  }
}
</style>
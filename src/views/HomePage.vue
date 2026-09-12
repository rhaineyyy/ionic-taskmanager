<template>
  <ion-page>

    <ion-header>
      <ion-toolbar>

        <ion-title>
          Daily Task Manager
        </ion-title>

      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">

      <!-- Welcome -->
      <div class="welcome-section">

        <div>
          <p class="small-text">
            Welcome back!
          </p>

          <h1>
            Manage Your Tasks
          </h1>

          <p class="description">
            Organize your daily activities and
            stay on top of your responsibilities.
          </p>
        </div>

      </div>


      <!-- Task Summary -->
      <TaskSummary
        :tasks="tasks"
      />


      <!-- Add Task -->
      <TaskForm
        @add-task="addTask"
      />


      <!-- Task List -->
      <TaskList
        :tasks="tasks"
        @edit-task="editTask"
        @delete-task="deleteTask"
        @toggle-status="toggleStatus"
      />


      <!-- Edit Modal -->
      <ion-modal
        :is-open="showEditModal"
        @didDismiss="closeEditModal"
      >

        <ion-header>
          <ion-toolbar>

            <ion-title>
              Edit Task
            </ion-title>

            <ion-buttons slot="end">
              <ion-button @click="closeEditModal">
                Close
              </ion-button>
            </ion-buttons>

          </ion-toolbar>
        </ion-header>


        <ion-content class="ion-padding">

          <ion-item>
            <ion-label position="stacked">
              Task Title
            </ion-label>

            <ion-input
              v-model="editingTask.title"
              type="text"
            />
          </ion-item>


          <ion-item>
            <ion-label position="stacked">
              Description
            </ion-label>

            <ion-textarea
              v-model="editingTask.description"
              :auto-grow="true"
            />
          </ion-item>


          <ion-item>
            <ion-label position="stacked">
              Due Date
            </ion-label>

            <ion-input
              v-model="editingTask.dueDate"
              type="date"
            />
          </ion-item>


          <ion-item>
            <ion-label position="stacked">
              Priority
            </ion-label>

            <ion-select
              v-model="editingTask.priority"
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


          <ion-item>
            <ion-label position="stacked">
              Status
            </ion-label>

            <ion-select
              v-model="editingTask.status"
            >
              <ion-select-option value="Pending">
                Pending
              </ion-select-option>

              <ion-select-option value="Completed">
                Completed
              </ion-select-option>
            </ion-select>
          </ion-item>


          <ion-button
            expand="block"
            class="save-button"
            @click="saveEdit"
          >
            Save Changes
          </ion-button>

        </ion-content>

      </ion-modal>

    </ion-content>

  </ion-page>
</template>


<script setup lang="ts">
import { reactive, ref } from 'vue'

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
  IonLabel,
  IonInput,
  IonTextarea,
  IonSelect,
  IonSelectOption
} from '@ionic/vue'

import TaskForm from '../components/TaskForm.vue'
import TaskList from '../components/TaskList.vue'
import TaskSummary from '../components/TaskSummary.vue'


interface Task {
  id: number
  title: string
  description: string
  dueDate: string
  priority: string
  status: string
}


/*
|--------------------------------------------------------------------------
| TASK DATA
|--------------------------------------------------------------------------
*/

const tasks = ref<Task[]>([])


/*
|--------------------------------------------------------------------------
| ADD TASK
|--------------------------------------------------------------------------
*/

const addTask = (newTask: {
  title: string
  description: string
  dueDate: string
  priority: string
  status: string
}) => {

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


/*
|--------------------------------------------------------------------------
| DELETE TASK
|--------------------------------------------------------------------------
*/

const deleteTask = (id: number) => {

  const confirmed = confirm(
    'Are you sure you want to delete this task?'
  )

  if (!confirmed) {
    return
  }

  tasks.value = tasks.value.filter(
    task => task.id !== id
  )
}


/*
|--------------------------------------------------------------------------
| TOGGLE TASK STATUS
|--------------------------------------------------------------------------
*/

const toggleStatus = (id: number) => {

  const task = tasks.value.find(
    task => task.id === id
  )

  if (!task) {
    return
  }

  task.status =
    task.status === 'Completed'
      ? 'Pending'
      : 'Completed'
}


/*
|--------------------------------------------------------------------------
| EDIT TASK
|--------------------------------------------------------------------------
*/

const showEditModal = ref(false)

const editingTask = reactive<Task>({
  id: 0,
  title: '',
  description: '',
  dueDate: '',
  priority: 'Medium',
  status: 'Pending'
})


const editTask = (task: Task) => {

  editingTask.id = task.id

  editingTask.title = task.title

  editingTask.description = task.description

  editingTask.dueDate = task.dueDate

  editingTask.priority = task.priority

  editingTask.status = task.status

  showEditModal.value = true
}


/*
|--------------------------------------------------------------------------
| SAVE EDIT
|--------------------------------------------------------------------------
*/

const saveEdit = () => {

  if (!editingTask.title.trim()) {
    alert('Task title is required.')
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


/*
|--------------------------------------------------------------------------
| CLOSE EDIT MODAL
|--------------------------------------------------------------------------
*/

const closeEditModal = () => {
  showEditModal.value = false
}
</script>


<style scoped>
ion-content {
  --background: #f5f7fa;
}


/*
|--------------------------------------------------------------------------
| HEADER
|--------------------------------------------------------------------------
*/

ion-toolbar {
  --background: #ffffff;
  --color: #222;
}

ion-title {
  font-weight: 700;
}


/*
|--------------------------------------------------------------------------
| WELCOME
|--------------------------------------------------------------------------
*/

.welcome-section {
  padding: 25px 20px 10px;
}

.small-text {
  margin: 0;
  font-size: 14px;
  color: #777;
}

.welcome-section h1 {
  margin: 5px 0;
  font-size: 30px;
  font-weight: 800;
  color: #222;
}

.description {
  margin: 8px 0 0;
  color: #777;
  line-height: 1.5;
}


/*
|--------------------------------------------------------------------------
| EDIT FORM
|--------------------------------------------------------------------------
*/

ion-modal ion-item {
  margin-bottom: 12px;
  --padding-start: 0;
  --inner-padding-end: 0;
}

.save-button {
  margin-top: 25px;
  --border-radius: 12px;
  height: 48px;
}
</style>
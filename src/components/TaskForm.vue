<template>
  <ion-card class="task-form-card">
    <ion-card-header>
      <ion-card-title>Add New Task</ion-card-title>
      <ion-card-subtitle>
        Create a task and keep track of your daily activities.
      </ion-card-subtitle>
    </ion-card-header>

    <ion-card-content>
      <ion-item>
        <ion-label position="stacked">Task Title</ion-label>
        <ion-input
          v-model="task.title"
          type="text"
          placeholder="Enter task title"
        />
      </ion-item>

      <ion-item>
        <ion-label position="stacked">Description</ion-label>
        <ion-textarea
          v-model="task.description"
          placeholder="Enter task description"
          :auto-grow="true"
        />
      </ion-item>

      <ion-item>
        <ion-label position="stacked">Due Date</ion-label>
        <ion-input
          v-model="task.dueDate"
          type="date"
        />
      </ion-item>

      <ion-item>
        <ion-label position="stacked">Priority</ion-label>
        <ion-select
          v-model="task.priority"
          placeholder="Select priority"
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
        <ion-label position="stacked">Status</ion-label>
        <ion-select v-model="task.status">
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
        class="add-button"
        @click="addTask"
      >
        <ion-icon :icon="addOutline" slot="start" />
        Add Task
      </ion-button>
    </ion-card-content>
  </ion-card>
</template>

<script setup lang="ts">
import { reactive } from 'vue'
import {
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonItem,
  IonLabel,
  IonInput,
  IonTextarea,
  IonSelect,
  IonSelectOption,
  IonButton,
  IonIcon
} from '@ionic/vue'

import { addOutline } from 'ionicons/icons'

const emit = defineEmits<{
  (e: 'add-task', task: {
    title: string
    description: string
    dueDate: string
    priority: string
    status: string
  }): void
}>()

const task = reactive({
  title: '',
  description: '',
  dueDate: '',
  priority: 'Medium',
  status: 'Pending'
})

const addTask = () => {
  if (!task.title.trim()) {
    alert('Please enter a task title.')
    return
  }

  emit('add-task', {
    title: task.title,
    description: task.description,
    dueDate: task.dueDate,
    priority: task.priority,
    status: task.status
  })

  task.title = ''
  task.description = ''
  task.dueDate = ''
  task.priority = 'Medium'
  task.status = 'Pending'
}
</script>

<style scoped>
.task-form-card {
  margin: 16px;
  border-radius: 18px;
}

ion-card-title {
  font-size: 22px;
  font-weight: 700;
}

ion-item {
  margin-bottom: 8px;
  --padding-start: 0;
  --inner-padding-end: 0;
}

.add-button {
  margin-top: 20px;
  --border-radius: 12px;
  height: 48px;
}
</style>
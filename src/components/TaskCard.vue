<template>
  <ion-card class="task-card">

    <ion-card-header>
      <div class="task-header">

        <div>
          <ion-card-title>
            {{ task.title }}
          </ion-card-title>

          <ion-card-subtitle>
            {{ task.description || 'No description' }}
          </ion-card-subtitle>
        </div>

        <ion-badge :color="priorityColor">
          {{ task.priority }}
        </ion-badge>

      </div>
    </ion-card-header>

    <ion-card-content>

      <div class="task-info">
        <div>
          <strong>Due Date</strong>
          <span>{{ formattedDate }}</span>
        </div>

        <div>
          <strong>Status</strong>

          <ion-badge :color="statusColor">
            {{ task.status }}
          </ion-badge>
        </div>
      </div>

      <div class="task-actions">

        <ion-button
          fill="outline"
          size="small"
          @click="editTask"
        >
          <ion-icon
            :icon="createOutline"
            slot="start"
          />
          Edit
        </ion-button>

        <ion-button
          size="small"
          :color="task.status === 'Completed' ? 'warning' : 'success'"
          @click="toggleStatus"
        >
          <ion-icon
            :icon="
              task.status === 'Completed'
                ? refreshOutline
                : checkmarkCircleOutline
            "
            slot="start"
          />

          {{ task.status === 'Completed'
            ? 'Pending'
            : 'Complete'
          }}
        </ion-button>

        <ion-button
          fill="clear"
          color="danger"
          size="small"
          @click="deleteTask"
        >
          <ion-icon :icon="trashOutline" />
        </ion-button>

      </div>

    </ion-card-content>

  </ion-card>
</template>

<script setup lang="ts">
import { computed } from 'vue'

import {
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonBadge,
  IonButton,
  IonIcon
} from '@ionic/vue'

import {
  createOutline,
  trashOutline,
  checkmarkCircleOutline,
  refreshOutline
} from 'ionicons/icons'

interface Task {
  id: number
  title: string
  description: string
  dueDate: string
  priority: string
  status: string
}

const props = defineProps<{
  task: Task
}>()

const emit = defineEmits<{
  (e: 'edit-task', task: Task): void
  (e: 'delete-task', id: number): void
  (e: 'toggle-status', id: number): void
}>()

const priorityColor = computed(() => {
  if (props.task.priority === 'High') {
    return 'danger'
  }

  if (props.task.priority === 'Medium') {
    return 'warning'
  }

  return 'success'
})

const statusColor = computed(() => {
  return props.task.status === 'Completed'
    ? 'success'
    : 'medium'
})

const formattedDate = computed(() => {
  if (!props.task.dueDate) {
    return 'No due date'
  }

  return new Date(
    props.task.dueDate + 'T00:00:00'
  ).toLocaleDateString()
})

const editTask = () => {
  emit('edit-task', props.task)
}

const deleteTask = () => {
  emit('delete-task', props.task.id)
}

const toggleStatus = () => {
  emit('toggle-status', props.task.id)
}
</script>

<style scoped>
.task-card {
  margin: 12px 16px;
  border-radius: 18px;
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
}

.task-info {
  display: flex;
  justify-content: space-between;
  gap: 15px;
  margin-bottom: 18px;
}

.task-info div {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.task-info strong {
  font-size: 12px;
  color: #777;
  text-transform: uppercase;
}

.task-info span {
  font-size: 14px;
}

.task-actions {
  display: flex;
  align-items: center;
  gap: 5px;
  flex-wrap: wrap;
}

ion-card-title {
  font-size: 19px;
  font-weight: 700;
}
</style>
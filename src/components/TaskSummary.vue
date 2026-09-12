<template>
  <div class="summary-container">

    <div class="summary-card total">
      <ion-icon :icon="listOutline" />

      <div>
        <span>Total Tasks</span>
        <strong>{{ total }}</strong>
      </div>
    </div>

    <div class="summary-card pending">
      <ion-icon :icon="timeOutline" />

      <div>
        <span>Pending</span>
        <strong>{{ pending }}</strong>
      </div>
    </div>

    <div class="summary-card completed">
      <ion-icon :icon="checkmarkDoneOutline" />

      <div>
        <span>Completed</span>
        <strong>{{ completed }}</strong>
      </div>
    </div>

  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'

import {
  IonIcon
} from '@ionic/vue'

import {
  listOutline,
  timeOutline,
  checkmarkDoneOutline
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
  tasks: Task[]
}>()

const total = computed(() => {
  return props.tasks.length
})

const pending = computed(() => {
  return props.tasks.filter(
    task => task.status === 'Pending'
  ).length
})

const completed = computed(() => {
  return props.tasks.filter(
    task => task.status === 'Completed'
  ).length
})
</script>

<style scoped>
.summary-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  padding: 16px;
}

.summary-card {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 15px;
  border-radius: 16px;
  background: white;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

.summary-card ion-icon {
  font-size: 28px;
}

.summary-card div {
  display: flex;
  flex-direction: column;
}

.summary-card span {
  font-size: 11px;
  color: #777;
}

.summary-card strong {
  font-size: 22px;
}

.total ion-icon {
  color: #3880ff;
}

.pending ion-icon {
  color: #f4b400;
}

.completed ion-icon {
  color: #2dd36f;
}

@media (max-width: 600px) {
  .summary-container {
    grid-template-columns: 1fr;
  }
}
</style>
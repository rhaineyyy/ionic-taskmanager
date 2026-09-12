<template>
  <div class="task-list">

    <div class="list-header">
      <div>
        <h2>My Tasks</h2>
        <p>
          {{ tasks.length }} task{{ tasks.length !== 1 ? 's' : '' }}
        </p>
      </div>

      <ion-select
        v-model="filter"
        interface="popover"
        class="filter-select"
      >
        <ion-select-option value="All">
          All
        </ion-select-option>

        <ion-select-option value="Pending">
          Pending
        </ion-select-option>

        <ion-select-option value="Completed">
          Completed
        </ion-select-option>
      </ion-select>
    </div>

    <div v-if="filteredTasks.length === 0" class="empty-state">

      <ion-icon
        :icon="checkmarkDoneOutline"
      />

      <h3>No tasks found</h3>

      <p>
        Add a new task to get started.
      </p>

    </div>

    <TaskCard
      v-for="task in filteredTasks"
      :key="task.id"
      :task="task"
      @edit-task="editTask"
      @delete-task="deleteTask"
      @toggle-status="toggleStatus"
    />

  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

import {
  IonIcon,
  IonSelect,
  IonSelectOption
} from '@ionic/vue'

import {
  checkmarkDoneOutline
} from 'ionicons/icons'

import TaskCard from './TaskCard.vue'

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

const emit = defineEmits<{
  (e: 'edit-task', task: Task): void
  (e: 'delete-task', id: number): void
  (e: 'toggle-status', id: number): void
}>()

const filter = ref('All')

const filteredTasks = computed(() => {
  if (filter.value === 'All') {
    return props.tasks
  }

  return props.tasks.filter(
    task => task.status === filter.value
  )
})

const editTask = (task: Task) => {
  emit('edit-task', task)
}

const deleteTask = (id: number) => {
  emit('delete-task', id)
}

const toggleStatus = (id: number) => {
  emit('toggle-status', id)
}
</script>

<style scoped>
.task-list {
  margin-top: 25px;
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 16px;
}

.list-header h2 {
  margin: 0;
  font-size: 24px;
  font-weight: 700;
}

.list-header p {
  margin: 4px 0 0;
  color: #777;
  font-size: 14px;
}

.filter-select {
  width: 120px;
}

.empty-state {
  text-align: center;
  padding: 50px 20px;
  color: #777;
}

.empty-state ion-icon {
  font-size: 60px;
}

.empty-state h3 {
  margin-bottom: 5px;
  color: #333;
}
</style>
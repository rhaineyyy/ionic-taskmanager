<template>
  <div class="create-task-form">

    <!-- TITLE -->
    <div class="field-block">
      <label class="field-label">Task Title</label>
      <div class="input-wrap">
        <ion-input
          v-model="task.title"
          type="text"
          placeholder="Enter task title"
          class="field-input"
        />
      </div>
    </div>

    <!-- DESCRIPTION -->
    <div class="field-block">
      <label class="field-label">Description</label>
      <div class="input-wrap">
        <ion-textarea
          v-model="task.description"
          placeholder="Enter task description"
          :auto-grow="true"
          :rows="3"
          class="field-input field-textarea"
        />
      </div>
    </div>

    <!-- DUE DATE -->
    <div class="field-block">
      <label class="field-label">Due Date</label>
      <div class="date-display" @click="openDatePicker">
        <span :class="{ 'is-placeholder': !formattedDate }">
          {{ formattedDate || 'Select a date' }}
        </span>
        <div class="date-icon-chip">
          <ion-icon :icon="calendarOutline" />
        </div>
      </div>
      <ion-input
        ref="dateInputRef"
        v-model="task.dueDate"
        type="date"
        :min="today"
        class="hidden-date-input"
      />
    </div>

    <!-- PRIORITY -->
    <div class="field-block">
      <label class="field-label">Priority</label>
      <div class="category-chips">
        <button
          v-for="p in priorities"
          :key="p.value"
          class="chip"
          :class="{ active: task.priority === p.value }"
          @click="task.priority = p.value"
          type="button"
        >
          {{ p.label }}
        </button>
      </div>
    </div>

    <!-- STATUS NOTE -->
    <div class="status-note">
      <ion-icon :icon="informationCircleOutline" class="note-icon" />
      <p>
        New tasks start as <strong>Pending</strong>.
        You can mark them as Completed later.
      </p>
    </div>

    <!-- ADD BUTTON -->
    <ion-button expand="block" class="add-button" @click="addTask">
      <ion-icon :icon="addOutline" slot="start" />
      Create Task
    </ion-button>

  </div>
</template>

<script setup lang="ts">
import { reactive, ref, computed, nextTick } from 'vue'
import { IonInput, IonTextarea, IonButton, IonIcon } from '@ionic/vue'
import {
  addOutline, calendarOutline, informationCircleOutline
} from 'ionicons/icons'

const today = new Date().toISOString().split('T')[0]

const emit = defineEmits<{
  (e: 'add-task', task: {
    title: string
    description: string
    dueDate: string
    priority: string
    status: string
  }): void
  (e: 'validation-error', payload: { title: string; message: string }): void
}>()

const priorities = [
  { label: 'Low',    value: 'Low' },
  { label: 'Medium', value: 'Medium' },
  { label: 'High',   value: 'High' }
]

const task = reactive({
  title: '',
  description: '',
  dueDate: '',
  priority: 'Medium'
})

const dateInputRef = ref<any>(null)

const formattedDate = computed(() => {
  if (!task.dueDate) return ''
  const d = new Date(task.dueDate + 'T00:00:00')
  return d.toLocaleDateString(undefined, {
    month: 'short',
    day: 'numeric',
    year: 'numeric'
  })
})

const openDatePicker = async () => {
  await nextTick()
  const ionEl = dateInputRef.value?.$el as HTMLElement | undefined
  const nativeInput = ionEl?.querySelector('input') as HTMLInputElement | null
  if (!nativeInput) return

  if (typeof (nativeInput as any).showPicker === 'function') {
    try {
      ;(nativeInput as any).showPicker()
      return
    } catch { /* fall through */ }
  }
  nativeInput.focus()
  nativeInput.click()
}

const addTask = () => {
  if (!task.title.trim()) {
    emit('validation-error', {
      title: 'Missing Title',
      message: 'Please enter a task title before continuing.'
    })
    return
  }
  if (!task.dueDate) {
    emit('validation-error', {
      title: 'Missing Due Date',
      message: 'Please select a due date for this task.'
    })
    return
  }
  if (task.dueDate < today) {
    emit('validation-error', {
      title: 'Invalid Date',
      message: 'Please select today or a future date.'
    })
    return
  }

  emit('add-task', {
    title: task.title.trim(),
    description: task.description.trim(),
    dueDate: task.dueDate,
    priority: task.priority,
    status: 'Pending'
  })

  task.title = ''
  task.description = ''
  task.dueDate = ''
  task.priority = 'Medium'
}
</script>

<style scoped>
/* =========================================================
   FORM WRAPPER
========================================================= */
.create-task-form {
  padding: 8px 22px 28px;
  max-width: 640px;
  margin: 0 auto;
}

/* =========================================================
   FIELD BLOCK
========================================================= */
.field-block {
  margin-bottom: 26px;
}

.field-label {
  display: block;
  font-size: 0.75rem;
  font-weight: 800;
  color: #3b5e4c;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  margin-bottom: 10px;
}

/* =========================================================
   INPUT WRAP — soft outlined oval container
========================================================= */
.input-wrap {
  display: flex;
  align-items: center;
  background: #ffffff;
  border: 1.5px solid #d3e0d7;
  border-radius: 18px;
  padding: 4px 16px;
  transition: border-color 0.25s ease, box-shadow 0.25s ease, border-radius 0.25s ease;
}

.input-wrap:focus-within {
  border-color: #3b5e4c;
  border-radius: 20px;
  box-shadow: 0 0 0 4px rgba(59, 94, 76, 0.10);
}

.field-input {
  --background: transparent;
  --padding-start: 0;
  --padding-end: 0;
  --padding-top: 14px;
  --padding-bottom: 14px;
  --color: #1c2e24 !important;
  --placeholder-color: #a8b8ae;
  --placeholder-opacity: 1;
  font-size: 1rem;
  font-weight: 600;
  width: 100%;
}

.field-textarea {
  font-size: 0.92rem;
  font-weight: 500;
  line-height: 1.55;
  --padding-top: 12px;
  --padding-bottom: 12px;
}

/* Remove native border since we now use wrapper */
.field-input::part(native),
.field-textarea::part(native) {
  border: none !important;
  padding: 0 !important;
}

/* =========================================================
   DATE DISPLAY — oval container
========================================================= */
.date-display {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 12px 14px 18px;
  background: #ffffff;
  border: 1.5px solid #d3e0d7;
  border-radius: 18px;
  cursor: pointer;
  transition: border-color 0.25s ease, box-shadow 0.25s ease, border-radius 0.25s ease;
}

.date-display:hover {
  border-color: #3b5e4c;
  border-radius: 20px;
}

.date-display:active {
  transform: scale(0.995);
}

.date-display span {
  font-size: 1rem;
  font-weight: 700;
  color: #1c2e24;
}

.date-display span.is-placeholder {
  color: #a8b8ae;
  font-weight: 500;
}

.date-icon-chip {
  width: 34px;
  height: 34px;
  border-radius: 11px;
  background: linear-gradient(135deg, #3b5e4c, #557a66);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  box-shadow: 0 4px 10px -3px rgba(59, 94, 76, 0.5);
}

.date-icon-chip ion-icon {
  font-size: 16px;
  color: #ffffff;
}

.hidden-date-input {
  position: absolute;
  opacity: 0;
  pointer-events: none;
  height: 0;
  width: 0;
  overflow: hidden;
}

/* =========================================================
   PRIORITY CHIPS
========================================================= */
.category-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.chip {
  padding: 11px 24px;
  border-radius: 999px;
  border: 1.5px solid #d3e0d7;
  background: #ffffff;
  color: #557a66;
  font-size: 0.85rem;
  font-weight: 800;
  cursor: pointer;
  font-family: inherit;
  letter-spacing: 0.02em;
  transition: all 0.2s ease;
}

.chip:hover {
  border-color: #3b5e4c;
  color: #3b5e4c;
  transform: translateY(-1px);
}

.chip.active {
  background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  color: #ffffff;
  border-color: transparent;
  box-shadow: 0 10px 20px -8px rgba(59, 94, 76, 0.6);
}

.chip.active:hover {
  transform: translateY(-1px);
}

/* =========================================================
   STATUS NOTE
========================================================= */
.status-note {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  margin: 4px 0 4px 2px;
  padding: 12px 16px;
  background: #f4f7f5;
  border: 1px solid #e1e9e3;
  border-radius: 16px;
}

.status-note .note-icon {
  font-size: 18px;
  color: #3b5e4c;
  flex-shrink: 0;
  margin-top: 1px;
}

.status-note p {
  margin: 0;
  font-size: 0.8rem;
  color: #557a66;
  font-weight: 500;
  line-height: 1.5;
}

.status-note strong {
  color: #3b5e4c;
  font-weight: 800;
}

/* =========================================================
   ADD BUTTON
========================================================= */
.add-button {
  margin-top: 26px;
  --background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  --background-hover: linear-gradient(135deg, #314e3f 0%, #4a6f5b 100%);
  --background-activated: #284033;
  --color: #ffffff;
  --border-radius: 18px;
  --box-shadow: 0 12px 28px -8px rgba(59, 94, 76, 0.5);
  height: 56px;
  font-weight: 800;
  font-size: 1rem;
  letter-spacing: 0.03em;
  text-transform: uppercase;
}

.add-button ion-icon {
  font-size: 20px;
}

/* =========================================================
   MOBILE
========================================================= */
@media (max-width: 480px) {
  .create-task-form {
    padding: 8px 18px 24px;
  }

  .field-block {
    margin-bottom: 22px;
  }

  .date-display {
    padding: 12px 12px 12px 16px;
  }

  .chip {
    padding: 10px 20px;
    font-size: 0.82rem;
  }

  .add-button {
    height: 52px;
    font-size: 0.95rem;
  }
}
</style>
<template>
  <div
    class="task-row"
    :class="{
      'is-completed': task.status === 'Completed',
      'menu-open': menuOpen
    }"
  >
    <!-- ICON -->
    <div class="task-icon">
      <ion-icon :icon="documentTextOutline" />
    </div>

    <!-- INFO -->
    <div class="task-info">
      <h4 class="task-title">{{ task.title }}</h4>

      <p v-if="task.description" class="task-description">
        {{ task.description }}
      </p>

      <div class="task-meta">
        <!-- PRIORITY -->
        <span
          class="meta-badge priority-badge"
          :class="`priority-${task.priority.toLowerCase()}`"
        >
          <ion-icon :icon="flagOutline" />
          {{ task.priority }}
        </span>

        <!-- STATUS -->
        <span
          class="meta-badge status-badge"
          :class="task.status === 'Completed' ? 'status-completed' : 'status-pending'"
        >
          <ion-icon :icon="task.status === 'Completed' ? checkmarkCircle : timeOutline" />
          {{ task.status }}
        </span>

        <!-- DUE DATE -->
        <span class="meta-badge date-badge">
          <ion-icon :icon="calendarOutline" />
          {{ formattedDate }}
        </span>
      </div>
    </div>

    <!-- ⋮ MENU -->
    <div class="task-actions">
      <button class="more-btn" @click.stop="$emit('toggle-menu')">
        <ion-icon :icon="ellipsisVerticalOutline" />
      </button>

      <div v-if="menuOpen" class="dropdown">
        <button @click="$emit('edit')">
          <ion-icon :icon="createOutline" /> Edit
        </button>
        <button @click="$emit('toggle-status')">
          <ion-icon :icon="checkmarkCircleOutline" />
          {{ task.status === 'Pending' ? 'Mark Complete' : 'Mark Pending' }}
        </button>
        <button class="danger" @click="$emit('delete')">
          <ion-icon :icon="trashOutline" /> Delete
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { IonIcon } from '@ionic/vue'
import {
  documentTextOutline,
  flagOutline,
  checkmarkCircle,
  timeOutline,
  calendarOutline,
  ellipsisVerticalOutline,
  createOutline,
  checkmarkCircleOutline,
  trashOutline
} from 'ionicons/icons'

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
   PROPS
========================= */
const props = defineProps<{
  task: Task
  menuOpen: boolean
}>()

/* =========================
   EMITS
========================= */
defineEmits<{
  (e: 'edit'): void
  (e: 'delete'): void
  (e: 'toggle-status'): void
  (e: 'toggle-menu'): void
}>()

/* =========================
   FORMATTED DATE
========================= */
const formattedDate = computed(() => {
  if (!props.task.dueDate) return 'No due date'
  const dt = new Date(props.task.dueDate + 'T00:00:00')
  return dt.toLocaleDateString(undefined, {
    month: 'short',
    day: 'numeric',
    year: 'numeric'
  })
})
</script>

<style scoped>
/* ================= TASK ROW ================= */
.task-row {
  display: flex;
  align-items: flex-start;
  gap: 14px;
  padding: 18px 20px;
  background: #ffffff;
  border: 1px solid #e1e9e3;

  /* ✅ MORE OVAL / PILL-SHAPED CORNERS */
  border-radius: 28px;

  box-shadow: 0 4px 14px rgba(45, 69, 56, 0.05);
  transition: transform 0.2s ease, box-shadow 0.2s ease, border-radius 0.2s ease;
  position: relative;
  overflow: visible;
  z-index: 1;
}

.task-row:hover {
  transform: translateY(-2px);
  border-radius: 32px;
  box-shadow: 0 12px 24px -10px rgba(45, 69, 56, 0.2);
}

/* Raise the row with the open menu above other rows */
.task-row.menu-open {
  z-index: 100;
  border-radius: 28px;
}

.task-row.is-completed {
  background: #fafcfb;
  border-color: #d1ded5;
}

.task-row.is-completed .task-title {
  text-decoration: line-through;
  color: #6b8275;
}

/* ================= ICON ================= */
.task-icon {
  width: 46px;
  height: 46px;
  /* ✅ Fully oval icon chip */
  border-radius: 16px;
  background: linear-gradient(135deg, #3b5e4c 0%, #557a66 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  box-shadow: 0 6px 14px -6px rgba(59, 94, 76, 0.5);
  margin-top: 2px;
  transition: border-radius 0.2s ease;
}

.task-row:hover .task-icon {
  border-radius: 18px;
}

.task-icon ion-icon {
  font-size: 20px;
  color: #ffffff;
}

/* ================= INFO ================= */
.task-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.task-title {
  margin: 0;
  font-size: 0.98rem;
  font-weight: 800;
  color: #1c2e24;
  letter-spacing: -0.01em;
  line-height: 1.3;
  word-break: break-word;
}

.task-description {
  margin: 0;
  font-size: 0.78rem;
  font-weight: 500;
  color: #557a66;
  line-height: 1.45;
  word-break: break-word;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

/* ================= META BADGES ================= */
.task-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 4px;
}

.meta-badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 5px 11px;

  /* ✅ Fully oval badges */
  border-radius: 999px;

  font-size: 0.66rem;
  font-weight: 800;
  letter-spacing: 0.03em;
  text-transform: uppercase;
  border: 1px solid transparent;
  white-space: nowrap;
  transition: transform 0.15s ease;
}

.meta-badge:hover {
  transform: translateY(-1px);
}

.meta-badge ion-icon {
  font-size: 11px;
}

.priority-high   { background: #fef2f2; color: #dc2626; border-color: #fecaca; }
.priority-medium { background: #fffbeb; color: #b45309; border-color: #fde68a; }
.priority-low    { background: #f0fdf4; color: #16a34a; border-color: #bbf7d0; }

.status-pending  { background: #eaf0ec; color: #3b5e4c; border-color: #d1e0d6; }
.status-completed{ background: #dcfce7; color: #15803d; border-color: #bbf7d0; }

.date-badge      { background: #f4f7f5; color: #3b5e4c; border-color: #e1e9e3; }

/* ================= MENU ================= */
.task-actions {
  position: relative;
  flex-shrink: 0;
  margin-top: -2px;
}

.more-btn {
  background: transparent;
  border: none;
  width: 34px;
  height: 34px;
  /* ✅ Oval menu button */
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #6b8275;
  transition: background 0.15s ease, color 0.15s ease, border-radius 0.15s ease;
  font-family: inherit;
}

.more-btn:hover {
  background: #eaf0ec;
  color: #3b5e4c;
  border-radius: 14px;
}

.more-btn ion-icon {
  font-size: 20px;
}

/* ================= DROPDOWN ================= */
.dropdown {
  position: absolute;
  right: 0;
  top: 44px;
  min-width: 190px;
  background: #ffffff;
  border: 1px solid #e1e9e3;

  /* ✅ Rounded dropdown */
  border-radius: 20px;

  box-shadow:
    0 16px 32px -12px rgba(45, 69, 56, 0.25),
    0 4px 12px rgba(45, 69, 56, 0.08);
  padding: 8px;
  z-index: 9999;
  animation: dropdownIn 0.15s ease both;
}

@keyframes dropdownIn {
  from { opacity: 0; transform: translateY(-4px) scale(0.98); }
  to   { opacity: 1; transform: translateY(0) scale(1); }
}

.dropdown button {
  display: flex;
  align-items: center;
  gap: 10px;
  width: 100%;
  padding: 10px 14px;
  border: none;
  background: transparent;
  color: #2d4538;
  font-size: 0.85rem;
  font-weight: 700;

  /* ✅ Fully oval button rows */
  border-radius: 12px;

  cursor: pointer;
  font-family: inherit;
  transition: background 0.15s ease, border-radius 0.15s ease;
  text-align: left;
}

.dropdown button:hover {
  background: #f4f7f5;
  border-radius: 14px;
}

.dropdown button ion-icon {
  font-size: 17px;
  color: #3b5e4c;
}

.dropdown button.danger {
  color: #dc2626;
}

.dropdown button.danger ion-icon {
  color: #dc2626;
}

.dropdown button.danger:hover {
  background: #fef2f2;
}

/* ================= MOBILE ================= */
@media (max-width: 600px) {
  .task-row {
    padding: 16px 18px;
    border-radius: 24px;
  }

  .task-title { font-size: 0.92rem; }

  .meta-badge {
    font-size: 0.62rem;
    padding: 4px 9px;
  }

  .task-icon {
    width: 42px;
    height: 42px;
    border-radius: 14px;
  }
}
</style>
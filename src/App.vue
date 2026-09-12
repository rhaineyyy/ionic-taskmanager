<template>
  <ion-app>
    <!-- LOADING / SPLASH SCREEN -->
    <div v-if="isLoading" class="loading-screen">
      <div class="glow glow-1"></div>
      <div class="glow glow-2"></div>

      <div class="splash-content">
        <!-- CENTERED ICON BADGE -->
        <div class="logo-badge">
          <ion-icon :icon="checkboxOutline" class="app-logo" />
        </div>

        <!-- TITLE & SUBTITLE -->
        <h1 class="app-title">Daily Task Manager</h1>
        <p class="app-subtitle">Preparing your task records...</p>

        <!-- LARGE PERCENTAGE DISPLAY -->
        <div class="percentage-display">{{ progress }}%</div>

        <!-- PROGRESS BAR -->
        <div class="progress-bar-container">
          <div class="progress-bar-fill" :style="{ width: progress + '%' }"></div>
        </div>

        <!-- BOTTOM STATUS TEXT -->
        <p class="status-text">Loading task information...</p>
      </div>
    </div>

    <!-- MAIN ROUTER OUTLET -->
    <ion-router-outlet v-else />
  </ion-app>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { IonApp, IonRouterOutlet, IonIcon } from '@ionic/vue'
import { checkboxOutline } from 'ionicons/icons'

/* =========================
   STATE
========================= */
const isLoading = ref(true)
const progress = ref(0)

/* =========================
   LOADING ANIMATION
========================= */
onMounted(() => {
  const DURATION = 6000         // 6 seconds total
  const TICK = 50               // update every 50ms
  const TOTAL_TICKS = DURATION / TICK
  let tick = 0

  const timer = setInterval(() => {
    tick++
    // Compute percentage based on elapsed ticks
    progress.value = Math.min(100, Math.round((tick / TOTAL_TICKS) * 100))

    if (tick >= TOTAL_TICKS) {
      clearInterval(timer)
      progress.value = 100
      // Small pause at 100% before hiding
      setTimeout(() => {
        isLoading.value = false
      }, 400)
    }
  }, TICK)
})
</script>

<style scoped>
/* =========================================================
   FULLSCREEN SAGE GREEN GRADIENT SPLASH SCREEN
========================================================= */
.loading-screen {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100vh;
  height: 100dvh;
  background: linear-gradient(160deg, #2d4538 0%, #3b5e4c 40%, #557a66 75%, #739882 100%);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 24px;
  box-sizing: border-box;
  z-index: 9999;
  overflow: hidden;
}

/* ================= AMBIENT GLOWS ================= */
.glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(60px);
  opacity: 0.5;
  pointer-events: none;
}

.glow-1 {
  width: 300px; height: 300px;
  background: #739882;
  top: -100px; right: -80px;
  animation: pulse 8s ease-in-out infinite;
}

.glow-2 {
  width: 260px; height: 260px;
  background: #3b5e4c;
  bottom: -100px; left: -60px;
  animation: pulse 10s ease-in-out infinite reverse;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); opacity: 0.5; }
  50%      { transform: scale(1.2); opacity: 0.7; }
}

/* ================= CONTENT ================= */
.splash-content {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  width: 100%;
  max-width: 320px;
  z-index: 1;
}

/* ================= LOGO ================= */
.logo-badge {
  width: 84px; height: 84px;
  background: rgba(255, 255, 255, 0.18);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  border-radius: 26px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 28px;
  border: 1px solid rgba(255, 255, 255, 0.35);
  box-shadow:
    0 12px 32px rgba(0, 0, 0, 0.18),
    inset 0 1px 0 rgba(255, 255, 255, 0.4);
  animation: floatY 4s ease-in-out infinite;
}

@keyframes floatY {
  0%, 100% { transform: translateY(0); }
  50%      { transform: translateY(-8px); }
}

.app-logo {
  font-size: 46px;
  color: #ffffff;
  filter: drop-shadow(0 2px 6px rgba(0, 0, 0, 0.25));
}

/* ================= TYPOGRAPHY ================= */
.app-title {
  font-size: 1.5rem;
  font-weight: 800;
  color: #ffffff;
  letter-spacing: -0.02em;
  margin: 0 0 6px 0;
  text-shadow: 0 2px 12px rgba(0, 0, 0, 0.2);
}

.app-subtitle {
  font-size: 0.875rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.85);
  margin: 0 0 32px 0;
}

/* ================= PERCENTAGE ================= */
.percentage-display {
  font-size: 2.5rem;
  font-weight: 800;
  color: #ffffff;
  margin-bottom: 14px;
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.03em;
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);
}

/* ================= PROGRESS BAR ================= */
.progress-bar-container {
  width: 100%;
  height: 8px;
  background: rgba(255, 255, 255, 0.22);
  border-radius: 999px;
  overflow: hidden;
  margin-bottom: 14px;
  backdrop-filter: blur(4px);
  box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1);
}

.progress-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #ffffff, #e8f5e9);
  border-radius: 999px;
  transition: width 0.08s linear;
  box-shadow: 0 0 14px rgba(255, 255, 255, 0.9);
}

/* ================= STATUS TEXT ================= */
.status-text {
  font-size: 0.75rem;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.8);
  margin: 0;
  letter-spacing: 0.02em;
}

/* ================= SMALL SCREENS ================= */
@media (max-width: 360px) {
  .logo-badge { width: 70px; height: 70px; border-radius: 22px; margin-bottom: 20px; }
  .app-logo { font-size: 38px; }
  .app-title { font-size: 1.3rem; }
  .percentage-display { font-size: 2rem; }
}
</style>
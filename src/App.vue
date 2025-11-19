<script setup>
import { onMounted, ref, watch } from 'vue';
import PlanBuilder from './components/PlanBuilder.vue';
import PlanTimer from './components/PlanTimer.vue';
import PomodoroTimer from './components/PomodoroTimer.vue';
import SettingsPanel from './components/SettingsPanel.vue';

// --- CONFIGURACIÓN DE ESTADO ---
const sessionDuration = ref(25);
const breakDuration = ref(5);
const alarm = ref('bell');
const volume = ref(0.8);
const muted = ref(false);
const settingsVersion = ref(0);

const productivePlan = ref([
  { type: 'session', duration: 25 },
  { type: 'break', duration: 5 },
  { type: 'session', duration: 25 },
  { type: 'break', duration: 5 },
  { type: 'session', duration: 25 },
  { type: 'break', duration: 15 }
]);

const LS = {
  session: 'pomodoro_session_duration',
  break: 'pomodoro_break_duration',
  alarm: 'pomodoro_alarm',
  volume: 'pomodoro_volume',
  muted: 'pomodoro_muted',
};

// --- WATCHERS PARA GUARDAR CONFIGURACIÓN ---
watch(sessionDuration, v => localStorage.setItem(LS.session, String(v)));
watch(breakDuration, v => localStorage.setItem(LS.break, String(v)));
watch(alarm, v => localStorage.setItem(LS.alarm, v));
watch(volume, v => localStorage.setItem(LS.volume, String(v)));
watch(muted, v => localStorage.setItem(LS.muted, String(v)));

onMounted(() => {
  const s = Number(localStorage.getItem(LS.session));
  const b = Number(localStorage.getItem(LS.break));
  const a = localStorage.getItem(LS.alarm);
  const v = Number(localStorage.getItem(LS.volume));
  const m = localStorage.getItem(LS.muted);

  if (!Number.isNaN(s) && s > 0) sessionDuration.value = s;
  if (!Number.isNaN(b) && b > 0) breakDuration.value = b;
  if (a) alarm.value = a;
  if (!Number.isNaN(v)) volume.value = v;
  if (m !== null) muted.value = m === 'true';
});

function onSettingsApply() {
  settingsVersion.value++;
}
</script>

<template>
  <div id="app" class="main-layout">
    <header class="app-header">
      <img src="@/assets/logo.png" alt="Logo" class="app-logo" />
      <h1>Pomodoro</h1>
    </header>

    <main class="mixer-main">
      <div class="mixer-center">
        <!-- FILA 1: Sesión Normal + Configuración -->
        <div class="mixer-row-container">
          <div class="mixer-timer">
            <PomodoroTimer
              :session-duration="sessionDuration"
              :break-duration="breakDuration"
              :alarm="alarm"
              :volume="volume"
              :muted="muted"
              :settings-version="settingsVersion"
              timer-title="Normal"
            />
          </div>

          <div class="mixer-config">
            <SettingsPanel
              :sessionDuration="sessionDuration"
              :breakDuration="breakDuration"
              :alarm="alarm"
              :volume="volume"
              :muted="muted"
              @update:sessionDuration="val => sessionDuration = val"
              @update:breakDuration="val => breakDuration = val"
              @update:alarm="val => alarm = val"
              @update:volume="val => volume = val"
              @update:muted="val => muted = val"
              @apply="onSettingsApply"
            />
          </div>
        </div>

        <!-- FILA 2: Plan Productivo + Configuración -->
        <div class="mixer-row-container">
          <div class="mixer-timer">
            <PlanTimer
              :plan="productivePlan"
              :alarm="alarm"
              :volume="volume"
              :muted="muted"
              timer-title="Plan Productivo"
            />
          </div>

          <div class="mixer-config">
            <PlanBuilder
              :plan="productivePlan"
              @update:plan="val => productivePlan = val"
            />
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
.main-layout { 
  width: 100%; 
  background-color: #fd9330;
  font-family: 'Quicksand', sans-serif;
}
.app-header {
  width: 100%;
  padding: 20px 16px 0;
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.app-logo {
  width: 95px;
  height: auto;
  margin-bottom: 6px;
}

.app-header h1 {
  margin: 0;
  font-size: 1.6rem;
}

.mixer-main {
  display: flex;
  flex-direction: column;
  gap: 15px;
  max-width: 100%;
  margin: 20px auto;
  padding: 0 8px;
  box-sizing: border-box;
  min-height: 100vh;
}

.mixer-center {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.mixer-row-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  align-items: center;
  min-height: 400px;
}

.mixer-timer {
  display: flex;
  justify-content: center;
  align-items: center;
  transform: scale(0.9735);
  transform-origin: center;
}

.mixer-config {
  display: flex;
  justify-content: center;
  align-items: center;
  transform: scale(0.9735);
  transform-origin: center;
}

/* RESPONSIVE */
@media (max-width: 1200px) {
  .mixer-row-container {
    grid-template-columns: 1fr;
    gap: 15px;
    min-height: auto;
  }
  .mixer-timer,
  .mixer-config {
    transform: scale(0.975);
  }
}

@media (max-width: 768px) {
  .mixer-row-container {
    grid-template-columns: 1fr;
  }
  .mixer-timer,
  .mixer-config {
    transform: scale(0.78);
  }
}
</style>

<template>
  <div class="plan-timer" :class="{ running: isRunning }">
    <div class="display">
      <h2>{{ currentCycleLabel }}</h2>
      <div class="cycle-info" :class="{'is-break': currentCycle.type === 'break', 'is-session': currentCycle.type === 'session'}">
        Ciclo {{ currentCycleIndex + 1 }} de {{ plan.length }}
      </div>
      <div class="time" :class="{ pulse: isRunning, 'break-mode': currentCycle.type === 'break' }">
        <div class="time-text">{{ formattedTime }}</div>
      </div>

      <div class="controls">
        <button class="primary-btn" @click="toggle">{{ isRunning ? 'Pausa' : 'Iniciar' }}</button>
        <button class="secondary-btn" @click="reset">Reset</button>
        <button class="secondary-btn" @click="nextCycle">Siguiente</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';

const props = defineProps({
  plan: { type: Array, default: () => [
    { type: 'session', duration: 25 },
    { type: 'break', duration: 5 },
    { type: 'session', duration: 25 },
    { type: 'break', duration: 5 },
    { type: 'session', duration: 25 },
    { type: 'break', duration: 15 }
  ]},
  alarm: { type: String, default: 'bell' },
  volume: { type: Number, default: 0.8 },
  muted: { type: Boolean, default: false }
});

const isRunning = ref(false);
const currentCycleIndex = ref(0);
const timeLeft = ref(0);
let intervalId = null;

watch(() => props.plan, (newPlan) => {
    // Si el plan cambia, reiniciamos el temporizador para aplicar el nuevo plan
    reset();
}, { deep: true });

onMounted(() => {
  if (props.plan.length > 0) {
    timeLeft.value = Math.round(props.plan[0].duration * 60);
  }
});

onUnmounted(() => clearInterval(intervalId));

const currentCycle = computed(() => props.plan[currentCycleIndex.value] || {});
const currentCycleLabel = computed(() => {
  const cycle = currentCycle.value;
  return cycle.type === 'session' ? 'Sesión' : 'Descanso';
});

const minutes = computed(() => Math.floor(timeLeft.value / 60));
const seconds = computed(() => Math.floor(timeLeft.value % 60));
const formattedTime = computed(() => `${String(minutes.value).padStart(2,'0')}:${String(seconds.value).padStart(2,'0')}`);

function tick() {
  if (timeLeft.value > 0) {
    timeLeft.value--;
  } else {
    playAlarm();
    nextCycle();
  }
}

function start() {
  if (intervalId) return;
  isRunning.value = true;
  intervalId = setInterval(tick, 1000);
}

function pause() {
  isRunning.value = false;
  clearInterval(intervalId);
  intervalId = null;
}

function toggle() {
  isRunning.value ? pause() : start();
}

function nextCycle() {
  pause();
  if (currentCycleIndex.value < props.plan.length - 1) {
    currentCycleIndex.value++;
    timeLeft.value = Math.round(props.plan[currentCycleIndex.value].duration * 60);
    start(); // Inicia el siguiente ciclo
  } else {
    // Plan terminado, resetear y parar
    currentCycleIndex.value = 0;
    timeLeft.value = Math.round((props.plan[0] ? props.plan[0].duration : 25) * 60);
  }
}

function reset() {
  pause();
  currentCycleIndex.value = 0;
  timeLeft.value = Math.round((props.plan[0] ? props.plan[0].duration : 25) * 60);
}

function playAlarm() {
  if (props.muted) return;
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const o = ctx.createOscillator();
    const g = ctx.createGain();

    let freq = 660, duration = 400, type = 'sine';

    switch(props.alarm) {
      case 'bell': freq = 660; duration = 400; break;
      case 'chime': freq = 880; duration = 400; break;
      case 'digital': freq = 1000; type = 'square'; duration = 400; break;
      case 'dog': freq = 800; duration = 300; break;
      case 'cat': freq = 1200; duration = 200; break;
      case 'rooster': freq = 1500; type = 'sawtooth'; duration = 600; break;
      case 'platypus': freq = 550; type = 'triangle'; duration = 500; break;
      case 'none': ctx.close(); return;
    }

    o.type = type;
    o.frequency.value = freq;
    g.gain.value = Math.max(0, Math.min(1, props.volume));
    o.connect(g);
    g.connect(ctx.destination);
    o.start();
    setTimeout(() => { o.stop(); ctx.close(); }, duration);
  } catch (e) {
    console.warn('Alarma: WebAudio no disponible', e);
  }
}
</script>

<style scoped>
.plan-timer { text-align:center; padding:20px; }
.display { width: 100%; max-width: 420px; background: #fff; border-radius:14px; padding:22px; box-shadow: 
    8px 8px 16px rgba(0, 0, 0, 0.1), 
    -8px -8px 16px rgba(255, 255, 255, 0.6); }

h2 {
    color: var(--color-text, #333); 
    transition: color 0.3s;
}

.cycle-info { 
    font-size: 0.9rem; 
    font-weight: 600; 
    margin-bottom: 8px; 
    transition: color 0.3s;
}

/* Color del texto de la información del ciclo */
.cycle-info.is-session {
    color: var(--color-pomodoro, #FF5722);
}
.cycle-info.is-break {
    color: var(--color-break, #2196F3);
}

.time { 
    margin: 14px auto; 
    width: 240px; 
    height: 240px; 
    display:grid; 
    place-items:center; 
    border-radius:50%; 
    position:relative; 
    transition: all 0.3s;
}

/* Fondo del reloj: Usa el color Pomodoro para sesión, color Break para descanso */
.time:not(.break-mode) {
    background: conic-gradient(var(--color-pomodoro, #FF5722) 0deg, rgba(255,87,34,0.08) 0deg);
}

.time.break-mode {
    background: conic-gradient(var(--color-break, #2196F3) 0deg, rgba(33,150,243,0.08) 0deg);
}

.time .time-text { font-size:3.25rem; font-weight:800; color:var(--color-text, #212121); }
.controls { margin-top: 16px; display:flex; justify-content:center; gap:10px; flex-wrap: wrap; }
.controls button { 
    padding:8px 12px; 
    border-radius:10px; 
    cursor:pointer; 
    border: none; 
    font-weight: 700; 
    transition: transform 0.2s, background 0.3s;
}

/* Botones con los colores del tema */
.controls button:first-child { 
    background: var(--color-pomodoro, #FF5722); 
    color: #fff;
}
.controls button:not(:first-child) {
    background: var(--color-break, #2196F3); 
    color: #fff;
}

.controls button:hover { transform: translateY(-2px); }
.pulse { animation: pulse 1s infinite; }
@keyframes pulse {
  0% { box-shadow: 0 12px 30px rgba(0, 0, 0, 0.06); }
  50% { box-shadow: 0 18px 40px rgba(0, 0, 0, 0.12); }
  100% { box-shadow: 0 12px 30px rgba(0, 0, 0, 0.06); }
}
</style>
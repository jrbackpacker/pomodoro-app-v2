<template>
  <div class="pomodoro" :class="{ running: isRunning }">
    <div class="display">
      <h2>{{ modeLabel }}</h2>
      <div class="time" :class="{ pulse: isRunning, 'break-mode': !isSession }">
        <div class="time-text">{{ formattedTime }}</div>
      </div>

      <div class="controls">
        <button class="primary-btn" @click="toggle">{{ isRunning ? 'Pausa' : 'Iniciar' }}</button>
        <button class="secondary-btn" @click="reset">Reset</button>
        <button class="secondary-btn" @click="switchMode(true)">Cambiar</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue';

const props = defineProps({
  sessionDuration: { type: Number, default: 25 },
  breakDuration: { type: Number, default: 5 },
  alarm: { type: String, default: 'bell' },
  volume: { type: Number, default: 0.8 },
  muted: { type: Boolean, default: false },
  settingsVersion: { type: Number, default: 0 }
});

const isSession = ref(true);
const isRunning = ref(false);
const timeLeft = ref(Math.round((props.sessionDuration || 25) * 60));
let intervalId = null;

// WATCH: cuando cambie settingsVersion, aplicar cambios
watch(() => props.settingsVersion, () => {
  pause();
  timeLeft.value = Math.round((isSession.value ? props.sessionDuration : props.breakDuration) * 60);
  console.log('Ajustes aplicados:', { isSession: isSession.value, timeLeft: timeLeft.value });
});

// WATCH: sessionDuration (cambios normales del padre)
watch(() => props.sessionDuration, (v) => {
  if (isSession.value && !isRunning.value) {
    timeLeft.value = Math.round((v || 25) * 60);
  }
});

// WATCH: breakDuration (cambios normales del padre)
watch(() => props.breakDuration, (v) => {
  if (!isSession.value && !isRunning.value) {
    timeLeft.value = Math.round((v || 5) * 60);
  }
});

onMounted(() => {
  timeLeft.value = Math.round((isSession.value ? props.sessionDuration : props.breakDuration) * 60);
});

onUnmounted(() => clearInterval(intervalId));

const minutes = computed(() => Math.floor(timeLeft.value / 60));
const seconds = computed(() => Math.floor(timeLeft.value % 60));
const formattedTime = computed(() => `${String(minutes.value).padStart(2,'0')}:${String(seconds.value).padStart(2,'0')}`);
const modeLabel = computed(() => isSession.value ? 'Sesión' : 'Descanso');

function tick() {
  if (timeLeft.value > 0) {
    timeLeft.value--;
  } else {
    playAlarm();
    switchMode(false);
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

function reset() {
  pause();
  timeLeft.value = Math.round((isSession.value ? props.sessionDuration : props.breakDuration) * 60);
}

function switchMode(manual = false) {
  pause();
  isSession.value = !isSession.value;
  timeLeft.value = Math.round((isSession.value ? props.sessionDuration : props.breakDuration) * 60);
  // Reanuda automáticamente si no es un cambio manual
  if (!manual) {
      start();
  }
}

function playAlarm() {
  if (props.muted) return;
  try {
    const ctx = new (window.AudioContext || window.webkitAudioContext)();
    const o = ctx.createOscillator();
    const g = ctx.createGain();

    let freq = 660;
    let duration = 400;
    let type = 'sine';

    switch(props.alarm) {
      case 'bell':
        freq = 660;
        duration = 400;
        break;
      case 'chime':
        freq = 880;
        duration = 400;
        break;
      case 'digital':
        freq = 1000;
        type = 'square';
        duration = 400;
        break;
      case 'dog':
        freq = 800;
        duration = 300;
        break;
      case 'cat':
        freq = 1200;
        duration = 200;
        break;
      case 'rooster':
        freq = 1500;
        duration = 600;
        type = 'sawtooth';
        break;
      case 'platypus':
        freq = 550;
        duration = 500;
        type = 'triangle';
        break;
      case 'none':
        ctx.close();
        return;
      default:
        freq = 660;
    }

    o.type = type;
    o.frequency.value = freq;
    g.gain.value = Math.max(0, Math.min(1, props.volume));
    o.connect(g);
    g.connect(ctx.destination);
    o.start();

    setTimeout(() => {
      o.stop();
      ctx.close();
    }, duration);
  } catch (e) {
    console.warn('Alarma: WebAudio no disponible', e);
  }
}
</script>

<style scoped>
.pomodoro { text-align:center; padding:20px; }
.display { 
  width: 100%; 
  max-width: 420px; 
  background: #fff; 
  border-radius:14px; 
  padding:22px; 
  box-shadow: 
    8px 8px 16px rgba(0, 0, 0, 0.1), 
    -8px -8px 16px rgba(255, 255, 255, 0.6); 
  transition: background 0.3s;
}

h2 {
    color: var(--color-text, #333); 
    transition: color 0.3s;
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


.time .time-text { 
  font-size:3.25rem; 
  font-weight:800; 
  color:var(--color-text, #212121); 
}
.controls { 
  margin-top: 16px; 
  display:flex; 
  justify-content:center; 
  gap:10px; 
  flex-wrap: wrap; 
}
.controls button { 
  padding:8px 12px; 
  border-radius:10px; 
  cursor:pointer; 
  border: none; 
  font-weight: 700;
  transition: transform 0.2s, background 0.3s;
}

/* Botones con los colores del tema */
.primary-btn { 
  background: var(--color-pomodoro, #FF5722); 
  color: #fff; 
}
.secondary-btn { 
  background: var(--color-break, #2196F3); 
  color: #fff;
}

.controls button:hover { 
  transform: translateY(-2px); 
}

.pulse { animation: pulse 1s infinite; }
/* La animación de pulse solo debe afectar la sombra */
@keyframes pulse {
  0% { box-shadow: 0 12px 30px rgba(0, 0, 0, 0.06); } 
  50% { box-shadow: 0 18px 40px rgba(0, 0, 0, 0.12); }
  100% { box-shadow: 0 12px 30px rgba(0, 0, 0, 0.06); }
}
</style>
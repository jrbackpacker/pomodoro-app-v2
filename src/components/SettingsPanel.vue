<template>
  <div class="settings-panel">
    <h2>Configurar sesión normal</h2>
    <p class="info-text">Los cambios se guardan al pulsar "Guardar cambios".</p>
    
    <div class="setting-group">
      <label for="session-duration">Sesión (min):</label>
      <input 
        id="session-duration" 
        type="number" 
        v-model.number="localSession"
        min="1"
        max="120"
      >
    </div>
    
    <div class="setting-group">
      <label for="break-duration">Descanso (min):</label>
      <input 
        id="break-duration"
        type="number"
        v-model.number="localBreak"
        min="1"
        max="60"
      >
    </div>



    <div class="setting-group">
      <label for="volume">Volumen:</label>
      <input 
        id="volume" 
        type="range" 
        v-model.number="localVolume"
        min="0"
        max="1"
        step="0.01"
      >
      <span class="volume-value">{{ Math.round(localVolume * 100) }}%</span>
    </div>

    <div class="setting-group">
      <label for="mute">Silenciar:</label>
      <input 
        id="mute" 
        type="checkbox" 
        v-model="localMuted"
      >
    </div>

    <div class="setting-actions">
      <button class="save-btn" @click="saveChanges">Guardar cambios</button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  sessionDuration: { type: Number, required: true },
  breakDuration: { type: Number, required: true },
  volume: { type: Number, default: 0.8 },
  muted: { type: Boolean, default: false }
});

const emit = defineEmits([
  'update:sessionDuration',
  'update:breakDuration',
  'update:volume',
  'update:muted',
  'apply'
]);

const localSession = ref(props.sessionDuration);
const localBreak = ref(props.breakDuration);
const localAlarm = ref(props.alarm);
const localVolume = ref(props.volume);
const localMuted = ref(props.muted);

watch(() => props.sessionDuration, v => { localSession.value = v; });
watch(() => props.breakDuration, v => { localBreak.value = v; });
watch(() => props.alarm, v => { localAlarm.value = v; });
watch(() => props.volume, v => { localVolume.value = v; });
watch(() => props.muted, v => { localMuted.value = v; });


function saveChanges() {
  const s = Math.max(1, Number(localSession.value) || 1);
  const b = Math.max(1, Number(localBreak.value) || 1);
  const a = localAlarm.value;
  const v = Math.max(0, Math.min(1, Number(localVolume.value) || 0.8));
  const m = localMuted.value;

  emit('update:sessionDuration', s);
  emit('update:breakDuration', b);
  emit('update:alarm', a);
  emit('update:volume', v);
  emit('update:muted', m);
  emit('apply');
}
</script>

<style scoped>

/* BOTONES DEL TIMBRE (NUEVO) */
.alarm-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.alarm-btn {
  padding: 6px 10px;
  border-radius: 8px;
  border: 2px solid #ccc;
  background: #fff;
  cursor: pointer;
  font-weight: 600;
  transition: 0.2s;
}

.alarm-btn:hover {
  border-color: var(--color-pomodoro, #FF5722);
}

.alarm-btn.active {
  background: var(--color-pomodoro, #FF5722);
  border-color: var(--color-pomodoro, #FF5722);
  color: white;
}




.settings-panel {
  background-color: #ffffff;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 
    8px 8px 16px rgba(0, 0, 0, 0.1), 
    -8px -8px 16px rgba(255, 255, 255, 0.6);
  width: 100%;
  max-width: 500px;
}

h2 {
  font-size: 1.6em;
  margin-bottom: 15px;
  text-align: center;
  color: #333;
  font-weight: 700;
}

.info-text {
  color: var(--color-pomodoro, #FF5722);
  text-align: center;
  margin-bottom: 20px;
  font-style: italic;
  font-size: 0.9em;
}

.setting-group {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 18px;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
  gap: 12px;
}

.setting-group:last-of-type {
  border-bottom: none;
  margin-bottom: 0;
}

label {
  font-weight: 600;
  color: #555;
  flex-shrink: 0;
  min-width: 100px;
}

input[type="number"],
input[type="range"] {
  width: 80px;
  padding: 8px;
  border: 2px solid #ccc;
  border-radius: 8px;
  text-align: center;
  font-size: 1.05em;
  font-weight: 600;
  transition: border-color 0.2s, box-shadow 0.2s;
  accent-color: var(--color-pomodoro, #FF5722);
}

input[type="range"] {
  width: 100px;
}

.volume-value {
  font-weight: 700;
  color: var(--color-pomodoro, #FF5722);
  min-width: 35px;
}

input[type="checkbox"] {
  width: 18px;
  height: 18px;
  accent-color: var(--color-pomodoro, #FF5722);
  cursor: pointer;
}

.setting-actions {
  margin-top: 18px;
  display: flex;
  justify-content: center;
}

.save-btn {
  background: var(--color-pomodoro, #FF5722);
  color: #fff;
  border: none;
  padding: 10px 18px;
  border-radius: 10px;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
  transition: transform 0.2s;
}
.save-btn:hover { 
    transform: translateY(-2px); 
    opacity: 0.9;
}

@media (max-width: 560px) {
  .setting-group {
    flex-direction: column;
    align-items: stretch;
  }
  label {
    min-width: auto;
    margin-bottom: 8px;
  }
  input[type="number"], input[type="range"] {
    width: 100%;
  }
}
</style>

<template>
  <div class="plan-builder">
    <h2>Configurar sesión productiva</h2>
    <p class="info-text">Añade sesiones y descansos a tu plan personalizado.</p>

    <div class="plan-list">
      <div v-for="(cycle, idx) in localPlan" :key="idx" class="cycle-item" :class="{'is-break': cycle.type === 'break', 'is-session': cycle.type === 'session'}">
        <div class="cycle-type">
          <select v-model="cycle.type" @change="updateCycle(idx)">
            <option value="session">Sesión</option>
            <option value="break">Descanso</option>
          </select>
        </div>
        <div class="cycle-duration">
          <input 
            type="number" 
            v-model.number="cycle.duration" 
            @change="updateCycle(idx)"
            min="1"
            max="120"
          >
          <span>min</span>
        </div>
        <button class="remove-btn" @click="removeCycle(idx)">✕</button>
      </div>
    </div>

    <div class="plan-actions">
      <button class="add-session-btn" @click="addCycle('session')">+ Sesión</button>
      <button class="add-break-btn" @click="addCycle('break')">+ Descanso</button>
    </div>

    <div class="plan-summary">
      <span v-if="localPlan.length === 0" class="empty">Plan vacío</span>
      <span v-else>{{ totalDuration }} min total • {{ sessionCount }} sesiones • {{ breakCount }} descansos</span>
    </div>

    <div class="plan-save">
      <button class="save-btn" @click="saveChanges">Guardar Plan</button>
      <button class="reset-btn" @click="resetToDefault">Restablecer</button>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue';

const props = defineProps({
  plan: { type: Array, required: true }
});

const emit = defineEmits(['update:plan']);

const localPlan = ref([]);

watch(() => props.plan, (newPlan) => {
  localPlan.value = JSON.parse(JSON.stringify(newPlan));
}, { immediate: true });

const totalDuration = computed(() => {
  return localPlan.value.reduce((acc, cycle) => acc + cycle.duration, 0);
});

const sessionCount = computed(() => {
  return localPlan.value.filter(c => c.type === 'session').length;
});

const breakCount = computed(() => {
  return localPlan.value.filter(c => c.type === 'break').length;
});

function addCycle(type) {
  const defaultDuration = type === 'session' ? 25 : 5;
  localPlan.value.push({ type, duration: defaultDuration });
}

function removeCycle(idx) {
  localPlan.value.splice(idx, 1);
}

function updateCycle(idx) {
  // Ya actualizado por v-model
}

function saveChanges() {
  if (localPlan.value.length === 0) {
    // Usamos console.error o un modal en lugar de alert
    console.error('El plan no puede estar vacío');
    return;
  }
  emit('update:plan', JSON.parse(JSON.stringify(localPlan.value)));
  console.log('Plan guardado:', localPlan.value);
}

function resetToDefault() {
  localPlan.value = [
    { type: 'session', duration: 25 },
    { type: 'break', duration: 5 },
    { type: 'session', duration: 25 },
    { type: 'break', duration: 5 },
    { type: 'session', duration: 25 },
    { type: 'break', duration: 15 }
  ];
}
</script>

<style scoped>
.plan-builder {
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

.plan-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 15px;
  max-height: 280px;
  overflow-y: auto;
  padding-right: 8px;
}

.cycle-item {
  display: flex;
  gap: 10px;
  align-items: center;
  padding: 12px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 1px solid #eee;
  transition: border-color 0.3s;
}

/* Resaltar el tipo de ciclo usando las variables del tema */
.cycle-item.is-session {
    border-left: 5px solid var(--color-pomodoro, #FF5722);
    background: rgba(255, 87, 34, 0.05); /* Usamos un color RGB genérico o del tema si es posible */
}
.cycle-item.is-break {
    border-left: 5px solid var(--color-break, #2196F3);
    background: rgba(33, 150, 243, 0.05); /* Usamos un color RGB genérico o del tema si es posible */
}

.cycle-type {
  flex: 1;
}

.cycle-type select {
  width: 100%;
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 6px;
  background: white;
  font-weight: 600;
  cursor: pointer;
}

.cycle-duration {
  display: flex;
  align-items: center;
  gap: 6px;
}

.cycle-duration input {
  width: 50px;
  padding: 6px;
  border: 1px solid #ccc;
  border-radius: 6px;
  text-align: center;
  font-weight: 600;
  accent-color: var(--color-pomodoro, #FF5722); 
}

.cycle-duration span {
  font-size: 0.9em;
  color: #666;
}

.remove-btn {
  background: #e53935;
  color: white;
  border: none;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 700;
  transition: transform 0.2s;
}

.remove-btn:hover {
  transform: scale(1.1);
}

.plan-actions {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.add-session-btn,
.add-break-btn {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 8px;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.2s;
}

/* Botón Añadir Sesión (color principal) */
.add-session-btn {
  background: var(--color-pomodoro, #4caf50); 
  color: white;
}

/* Botón Añadir Descanso (color secundario) */
.add-break-btn {
  background: var(--color-break, #2196f3); 
  color: white;
}

.add-session-btn:hover,
.add-break-btn:hover {
  transform: translateY(-2px);
}

.plan-summary {
  text-align: center;
  font-size: 0.95em;
  color: #666;
  margin-bottom: 15px;
  padding: 10px;
  background: #f0f0f0;
  border-radius: 8px;
}

.empty {
  color: #999;
  font-style: italic;
}

.plan-save {
  display: flex;
  gap: 10px;
}

.save-btn,
.reset-btn {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 8px;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.2s;
}

.save-btn {
  background: var(--color-pomodoro, #FF5722);
  color: white;
}

.reset-btn {
  background: #999;
  color: white;
}

.save-btn:hover,
.reset-btn:hover {
  transform: translateY(-2px);
}

@media (max-width: 560px) {
  .cycle-item {
    flex-wrap: wrap;
  }
  .plan-list {
    max-height: 400px;
  }
}
</style>
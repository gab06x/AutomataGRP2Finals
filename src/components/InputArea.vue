<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
    regexStr:    { type: String,  required: true },
    dfaResults:  { type: Array,   default: () => [] },   // Array of null | 'Valid' | 'Invalid' per row
    isRunningAll:{ type: Boolean, default: false }
})

const emit = defineEmits(['simulate-string', 'run-all', 'cancel-simulation', 'strings-changed', 'clear-all'])

const NUM_INPUTS = 5
const userInputs = ref(Array(NUM_INPUTS).fill(''))

// Broadcast the full array up so the parent can track all strings
watch(userInputs, (val) => emit('strings-changed', val.slice()), { deep: true })

// Compute regex badge (Valid / Invalid) based on the current regex
const regexResult = computed(() =>
  userInputs.value.map(input => {
    if (!props.regexStr) return null
    if (input === '') return null
    try {
      const cleanedRegex = props.regexStr.replace(/\s+/g, '')
      const jsRegexStr = '^' + cleanedRegex.split('+').join('|') + '$'
      const regex = new RegExp(jsRegexStr)
      return regex.test(input) ? 'Valid' : 'Invalid'
    } catch {
      return 'Error'
    }
  })
)

// DFA result comes from parent (after simulation finishes)
const dfaResult = (idx) => props.dfaResults[idx] ?? null

const simulate = (idx) => {
  if (userInputs.value[idx] !== '') {
    emit('simulate-string', { str: userInputs.value[idx], index: idx })
  }
}

const runAll = () => emit('run-all', userInputs.value.slice())
const cancelSim = () => emit('cancel-simulation')
const clearAll = () => {
  userInputs.value = Array(NUM_INPUTS).fill('')
  emit('clear-all')
}
</script>

<template>
  <div class="input-area-container">
    <div class="inputs-area">
      <div
        v-for="idx in NUM_INPUTS"
        :key="idx - 1"
        class="input-row"
        :class="{ 'row-active': isRunningAll && dfaResult(idx - 1) === null && userInputs[idx - 1] !== '' }"
      >
        <!-- Row number -->
        <span class="row-num">{{ idx }}</span>

        <!-- Input wrapper -->
        <div class="input-wrapper">
          <span class="input-prefix">›</span>
          <input
            v-model="userInputs[idx - 1]"
            :placeholder="`String ${idx}`"
            class="input-field"
            :disabled="isRunningAll"
            @keyup.enter="simulate(idx - 1)"
          />
        </div>

        <!-- Regex badge -->
        <span
          v-if="userInputs[idx - 1] !== '' && regexResult[idx - 1]"
          :class="['result-badge', regexResult[idx - 1] === 'Valid' ? 'valid' : 'invalid']"
        >
          {{ regexResult[idx - 1] }}
        </span>
        <span v-else class="result-badge placeholder-badge">—</span>

        <!-- DFA result badge -->
        <span
          :class="[
            'dfa-badge',
            dfaResult(idx - 1) === 'Valid'   ? 'dfa-valid'   :
            dfaResult(idx - 1) === 'Invalid' ? 'dfa-invalid' : 'dfa-pending'
          ]"
        >
          <template v-if="dfaResult(idx - 1)">
            <svg v-if="dfaResult(idx - 1) === 'Valid'" width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round"><polyline points="20 6 9 17 4 12"/></svg>
            <svg v-else width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3" stroke-linecap="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
            DFA {{ dfaResult(idx - 1) }}
          </template>
          <template v-else>DFA —</template>
        </span>

        <!-- Simulate button -->
        <button
          :disabled="isRunningAll || !userInputs[idx - 1]"
          @click="simulate(idx - 1)"
          class="simulate-btn"
        >
          Run
        </button>
      </div>
    </div>

    <!-- Footer buttons -->
    <div class="footer-btns">
      <button
        class="run-all-btn"
        :disabled="isRunningAll || userInputs.every(s => s === '')"
        @click="runAll"
      >
        <svg width="12" height="12" viewBox="0 0 24 24" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3"/></svg>
        Run All
      </button>

      <button
        class="cancel-btn"
        :disabled="!isRunningAll"
        @click="cancelSim"
      >
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"><rect x="4" y="4" width="16" height="16" rx="2"/></svg>
        Cancel
      </button>

      <button
        class="clear-all-btn"
        :disabled="isRunningAll || userInputs.every(s => s === '')"
        @click="clearAll"
      >
        <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14a2 2 0 0 1-2 2H8a2 2 0 0 1-2-2L5 6"/><path d="M10 11v6M14 11v6"/><path d="M9 6V4h6v2"/></svg>
        Clear All
      </button>
    </div>
  </div>
</template>

<style scoped>
.input-area-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.inputs-area {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.input-row {
  display: flex;
  align-items: center;
  gap: 6px;
  flex-wrap: wrap;
  padding: 2px 4px;
  border-radius: 4px;
  border: 1px solid transparent;
  transition: border-color 0.25s, background 0.25s;
}

.input-row.row-active {
  border-color: rgba(0, 229, 255, 0.2);
  background: rgba(0, 229, 255, 0.03);
}

.row-num {
  font-size: 10px;
  font-weight: 700;
  color: #2d4a6b;
  min-width: 14px;
  text-align: center;
  font-family: 'Space Mono', monospace;
  letter-spacing: 0.05em;
}

.input-wrapper {
  position: relative;
  flex: 1;
  min-width: 140px;
}

.input-prefix {
  position: absolute;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 13px;
  color: #2d4a6b;
  font-family: 'Space Mono', monospace;
  pointer-events: none;
  z-index: 1;
}

.input-field {
  width: 100%;
  padding: 5px 8px 5px 22px;
  border: 1px solid #1e2d3d;
  border-radius: 4px;
  font-size: 11px;
  background: #111827;
  transition: all 0.2s ease;
  font-family: 'Space Mono', monospace;
  color: #c8d0e0;
  box-sizing: border-box;
}

.input-field:focus {
  outline: none;
  border-color: #00e5ff;
  box-shadow: 0 0 0 2px rgba(0, 229, 255, 0.12), 0 0 10px rgba(0, 229, 255, 0.08);
  background: #0d1520;
  color: #e8edf5;
}

.input-field::placeholder { color: #2d4a6b; }
.input-field:disabled { opacity: 0.5; cursor: not-allowed; }

/* Regex result badge */
.result-badge {
  font-size: 9px;
  font-weight: 700;
  padding: 3px 6px;
  border-radius: 3px;
  text-transform: uppercase;
  min-width: 50px;
  text-align: center;
  white-space: nowrap;
  letter-spacing: 0.06em;
  flex-shrink: 0;
}

.placeholder-badge {
  background: transparent;
  color: #2d4a6b;
  border: 1px solid #1e2d3d;
}

.result-badge.valid {
  background: rgba(200, 255, 0, 0.1);
  color: #c8ff00;
  border: 1px solid rgba(200, 255, 0, 0.25);
  box-shadow: 0 0 6px rgba(200, 255, 0, 0.12);
}

.result-badge.invalid {
  background: rgba(255, 68, 68, 0.1);
  color: #ff6b6b;
  border: 1px solid rgba(255, 68, 68, 0.25);
}

/* DFA result badge */
.dfa-badge {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 9px;
  font-weight: 700;
  padding: 4px 8px;
  border-radius: 3px;
  text-transform: uppercase;
  min-width: 72px;
  text-align: center;
  justify-content: center;
  white-space: nowrap;
  letter-spacing: 0.08em;
  flex-shrink: 0;
  transition: all 0.3s ease;
}

.dfa-pending {
  background: transparent;
  color: #2d4a6b;
  border: 1px solid #1e2d3d;
}

.dfa-valid {
  background: rgba(16, 185, 129, 0.12);
  color: #10b981;
  border: 1px solid rgba(16, 185, 129, 0.3);
  box-shadow: 0 0 8px rgba(16, 185, 129, 0.15);
}

.dfa-invalid {
  background: rgba(239, 68, 68, 0.12);
  color: #ef4444;
  border: 1px solid rgba(239, 68, 68, 0.3);
}

/* Simulate button */
.simulate-btn {
  padding: 7px 14px;
  border-radius: 4px;
  border: 1px solid #00e5ff;
  background: transparent;
  color: #00e5ff;
  font-size: 10px;
  font-weight: 700;
  font-family: 'Space Mono', monospace;
  cursor: pointer;
  transition: all 0.2s ease;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  white-space: nowrap;
  flex-shrink: 0;
}

.simulate-btn:hover:not(:disabled) {
  background: rgba(0, 229, 255, 0.1);
  box-shadow: 0 0 10px rgba(0, 229, 255, 0.25);
}

.simulate-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

/* Footer buttons */
.footer-btns {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  padding-top: 8px;
  border-top: 1px solid #1a2535;
  margin-top: 8px;
}

.run-all-btn,
.cancel-btn,
.clear-all-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 6px 8px;
  border-radius: 4px;
  font-size: 10px;
  font-weight: 700;
  font-family: 'Space Mono', monospace;
  cursor: pointer;
  transition: all 0.2s ease;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  white-space: nowrap;
  flex: 1;
}

.clear-all-btn {
  flex-basis: 100%;
}

.run-all-btn {
  background: #00e5ff;
  border: 1px solid #00e5ff;
  color: #0a0a0f;
  box-shadow: 0 0 12px rgba(0, 229, 255, 0.3);
}

.run-all-btn:hover:not(:disabled) {
  background: #33eaff;
  border-color: #33eaff;
  box-shadow: 0 0 20px rgba(0, 229, 255, 0.5);
  transform: translateY(-1px);
}

.run-all-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  box-shadow: none;
  transform: none;
}

.cancel-btn {
  background: transparent;
  border: 1px solid rgba(239, 68, 68, 0.4);
  color: #ef4444;
}

.cancel-btn:hover:not(:disabled) {
  background: rgba(239, 68, 68, 0.1);
  border-color: #ef4444;
  box-shadow: 0 0 10px rgba(239, 68, 68, 0.2);
}

.cancel-btn:disabled {
  opacity: 0.25;
  cursor: not-allowed;
}

.clear-all-btn {
  background: transparent;
  border: 1px solid rgba(240, 165, 0, 0.4);
  color: #f0a500;
}

.clear-all-btn:hover:not(:disabled) {
  background: rgba(240, 165, 0, 0.1);
  border-color: #f0a500;
  box-shadow: 0 0 10px rgba(240, 165, 0, 0.2);
}

.clear-all-btn:disabled {
  opacity: 0.25;
  cursor: not-allowed;
}

@media (max-width: 768px) {
  .input-row { flex-wrap: wrap; gap: 6px; }
  .footer-btns { flex-wrap: wrap; }
  .run-all-btn, .cancel-btn, .clear-all-btn { flex: 1; justify-content: center; }
}
</style>
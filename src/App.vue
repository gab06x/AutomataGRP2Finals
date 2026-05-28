<script setup>
import { ref, computed } from 'vue'
import Problems from './components/Promblems.vue'
import InputArea from './components/InputArea.vue'
import NavBar from './components/NavBar.vue'
import Diagram from './components/DFAVisualization.vue'
import PDAVisualization from './components/PDAVisualization.vue'
import CFGVisualization from './components/CFGVisualization.vue'
import UserManual from './components/UserManual.vue'
import Footer from './components/Footer.vue'

const activeAutomata = ref('dfa')

const automata = ref('automata-theory project')

const problems = ref([
  {
    id: 1,
    label: 'Regex 1',
    regexStr: '(bab+bbb) b* a* (a*+b*) (ab)* (aba) (bab+aba)* bb (a+b)* (bab+aba) (a+b)*'
  },
  {
    id: 2,
    label: 'Regex 2',
    regexStr: '(1+0)* 0* 1* (111+00+101) (1+0)* (101+01+000) (1+0)* (101+000)*'
  }
])

const selectedProblemIndex = ref(-1)
const currentRegex = computed(() => {
    if (selectedProblemIndex.value === -1) return ''
    return problems.value[selectedProblemIndex.value]?.regexStr || ''
})

// --- Simulation state ---
const currentTestString = ref('')
const currentSimIndex   = ref(-1)
const simulationKey     = ref(0)
const isRunningAll      = ref(false)
const isSimulating      = ref(false)                // true when any simulation (single or all) is active
const isManualOpen      = ref(false)
const dfaResults        = ref(Array(5).fill(null))  // null | 'Valid' | 'Invalid'

// --- String tracking & history ---
const testStrings   = ref(Array(5).fill(''))        // live copies of all 5 inputs
const dfaTraces     = ref(Array(5).fill(null))       // stored { steps, accepted } per row
const activeViewIdx = ref(-1)                        // -1 = follow live sim; >=0 = viewing that row

// Sync strings from InputArea
const onStringsChanged = (strings) => {
  testStrings.value = [...strings]
}

// --- What to pass to the DFA for static replay (null = live animation mode) ---
const dfaViewTrace = computed(() => {
  const idx = activeViewIdx.value
  if (idx < 0) return null
  return dfaTraces.value[idx] || null
})

// --- What string the CFG should derive ---
const cfgTestString = computed(() => {
  const idx = activeViewIdx.value >= 0 ? activeViewIdx.value : currentSimIndex.value
  if (idx >= 0) return testStrings.value[idx] || ''
  return currentTestString.value
})

// Set view index (toggle off if same pill clicked)
const setViewIdx = (idx) => {
  if (isRunningAll.value) return
  activeViewIdx.value = activeViewIdx.value === idx ? -1 : idx
}

// Run a single string (from the "Run" button in a row)
const runSimulation = ({ str, index }) => {
  activeViewIdx.value = -1  // return to live mode
  dfaResults.value = dfaResults.value.map((v, i) => i === index ? null : v)
  currentTestString.value = str
  currentSimIndex.value   = index
  simulationKey.value++
  isSimulating.value = true
}

// ---- Run All logic ----
let runAllResolver = null

// Called by DFA when its simulation timer finishes
const onSimulationDone = ({ index, accepted, steps }) => {
  const results = [...dfaResults.value]
  results[index] = accepted ? 'Valid' : 'Invalid'
  dfaResults.value = results

  // Cache the full trace for static replay
  const traces = [...dfaTraces.value]
  traces[index] = { steps, accepted }
  dfaTraces.value = traces

  // If we're in a run-all sequence, resolve the promise to advance
  if (runAllResolver) {
    const resolve = runAllResolver
    runAllResolver = null
    resolve()
  } else {
    // Single run finished
    isSimulating.value = false
  }
}

const runAll = async (strings) => {
  if (isRunningAll.value) return
  isRunningAll.value = true
  isSimulating.value = true
  activeViewIdx.value = -1  // live mode during run-all
  // Reset all DFA results and traces
  dfaResults.value = Array(5).fill(null)
  dfaTraces.value  = Array(5).fill(null)

  for (let i = 0; i < strings.length; i++) {
    if (!isRunningAll.value) break   // cancelled
    const str = strings[i]
    if (str === '') continue         // skip empty rows

    currentTestString.value = str
    currentSimIndex.value   = i
    simulationKey.value++

    // Wait for DFA to emit simulation-done
    await new Promise(resolve => { runAllResolver = resolve })

    if (!isRunningAll.value) break   // cancelled after await
  }

  isRunningAll.value = false
  isSimulating.value = false
  runAllResolver     = null
}

const cancelSimulation = () => {
  isRunningAll.value = false
  isSimulating.value = false
  if (runAllResolver) {
    // Unblock the loop so it exits cleanly
    const resolve = runAllResolver
    runAllResolver  = null
    resolve()
  }
}

// Clear All — reset inputs, results, traces, and view state
const onClearAll = () => {
  dfaResults.value    = Array(5).fill(null)
  dfaTraces.value     = Array(5).fill(null)
  testStrings.value   = Array(5).fill('')
  activeViewIdx.value = -1
  currentTestString.value = ''
  currentSimIndex.value   = -1
  isSimulating.value      = false
}
</script>

<template>
  <NavBar @open-manual="isManualOpen = true" />

  <div class="app-shell">
    <!-- ===== LEFT SIDEBAR ===== -->
    <aside class="sidebar">

      <div class="sidebar-section">
        <div class="sidebar-label">Regex Selection <span v-if="isSimulating" class="label-lock">🔒</span></div>
        <Problems
          :problems="problems"
          v-model="selectedProblemIndex"
          :disabled="isSimulating"
        />
      </div>

      <div class="sidebar-divider"></div>

      <template v-if="selectedProblemIndex !== -1">

        <!-- Test Terminal moved to Sidebar -->
        <div class="sidebar-section">
          <div class="sidebar-label terminal-header-sidebar">
            <span>Test Terminal</span>
            <span v-if="isRunningAll" class="terminal-running-badge">● Running All…</span>
          </div>
          <InputArea
            :regexStr="currentRegex"
            :dfaResults="dfaResults"
            :isRunningAll="isRunningAll"
            @simulate-string="runSimulation"
            @run-all="runAll"
            @cancel-simulation="cancelSimulation"
            @strings-changed="onStringsChanged"
            @clear-all="onClearAll"
          />
        </div>
      </template>
    </aside>

    <!-- ===== MAIN AREA ===== -->
    <main class="main-panel">

      <!-- Tabs bar -->
      <div class="tab-bar">
        <button
          class="qa-item"
          :class="{ active: activeAutomata === 'dfa' }"
          @click="activeAutomata = 'dfa'"
        >
          <span class="qa-icon">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none"><rect x="3" y="3" width="8" height="8" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="13" y="3" width="8" height="8" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="3" y="13" width="8" height="8" rx="1" stroke="currentColor" stroke-width="1.5"/><rect x="13" y="13" width="8" height="8" rx="1" stroke="currentColor" stroke-width="1.5"/></svg>
          </span>
          STATE MAP (DFA)
        </button>
        <button
          class="qa-item"
          :class="{ active: activeAutomata === 'cfg' }"
          @click="activeAutomata = 'cfg'"
        >
          <span class="qa-icon">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none"><path d="M4 6h16M4 10h16M4 14h10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>
          </span>
          CFG TREE
        </button>
        <button
          class="qa-item"
          :class="{ active: activeAutomata === 'pda' }"
          @click="activeAutomata = 'pda'"
        >
          <span class="qa-icon">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none"><path d="M4 4h16v3H4zM4 10h16v3H4zM4 16h10v3H4z" stroke="currentColor" stroke-width="1.5"/></svg>
          </span>
          EXECUTION LOGS (PDA)
        </button>
      </div>

      <!-- String selector bar (shown when strings are available) -->
      <div
        class="str-selector-bar"
        v-if="selectedProblemIndex !== -1 && activeAutomata !== 'manual' && testStrings.some(s => s)"
      >
        <span class="str-sel-label">View:</span>

        <!-- Current Sim pill -->
        <button
          class="str-pill"
          :class="{ 'sp-active': activeViewIdx === -1 }"
          @click="activeViewIdx = -1"
          :disabled="isRunningAll"
          title="Follow the currently running simulation"
        >
          <span class="sp-live-dot" :class="{ 'sp-live-dot--anim': isRunningAll || currentSimIndex >= 0 }"></span>
          Current Sim
        </button>

        <div class="str-sel-sep"></div>

        <!-- One pill per non-empty string -->
        <template v-for="(str, idx) in testStrings" :key="idx">
          <button
            v-if="str"
            class="str-pill sp-numbered"
            :class="{
              'sp-active':   activeViewIdx === idx,
              'sp-valid':    dfaResults[idx] === 'Valid',
              'sp-invalid':  dfaResults[idx] === 'Invalid',
              'sp-running':  currentSimIndex === idx && isRunningAll
            }"
            @click="setViewIdx(idx)"
            :disabled="isRunningAll"
            :title="str"
          >
            <span class="sp-num">{{ idx + 1 }}</span>
            <span class="sp-str">{{ str.length > 9 ? str.slice(0, 9) + '…' : str }}</span>
            <span v-if="dfaResults[idx] === 'Valid'"   class="sp-dot sp-dot--ok">✓</span>
            <span v-else-if="dfaResults[idx] === 'Invalid'" class="sp-dot sp-dot--fail">✗</span>
            <span v-else-if="currentSimIndex === idx && isRunningAll" class="sp-dot sp-dot--spin">⟳</span>
          </button>
        </template>
      </div>
      <!-- Visualization canvas -->
      <div class="viz-canvas">
        <template v-if="selectedProblemIndex !== -1">
          <!-- DFA: kept in DOM at all times via v-show so simulation timer isn't killed by tab switches -->
          <div v-show="activeAutomata === 'dfa'" class="viz-inner">
            <Diagram
              :problemId="problems[selectedProblemIndex].id"
              :testString="currentTestString"
              :simKey="simulationKey"
              :simIndex="currentSimIndex"
              :viewTrace="dfaViewTrace"
              @simulation-done="onSimulationDone"
            />
          </div>
          <div v-show="activeAutomata === 'cfg'" class="viz-inner">
            <CFGVisualization
              :problemId="problems[selectedProblemIndex].id"
              :testString="cfgTestString"
            />
          </div>
          <div v-show="activeAutomata === 'pda'" class="viz-inner">
            <PDAVisualization
              :problemId="problems[selectedProblemIndex].id"
            />
          </div>
        </template>

        <!-- Empty state -->
        <div v-else class="empty-canvas">
          <div class="empty-icon">
            <svg width="48" height="48" viewBox="0 0 24 24" fill="none">
              <circle cx="5" cy="12" r="2.5" stroke="#1e3a5f" stroke-width="1.5"/>
              <circle cx="19" cy="5" r="2.5" stroke="#1e3a5f" stroke-width="1.5"/>
              <circle cx="19" cy="19" r="2.5" stroke="#1e3a5f" stroke-width="1.5"/>
              <path d="M7.5 12h5M14.5 7l4-2M14.5 17l4 2" stroke="#1e3a5f" stroke-width="1.5" stroke-linecap="round"/>
            </svg>
          </div>
          <div class="empty-text">Select a problem to begin simulation</div>
        </div>
      </div>

      <!-- User Manual Drawer Overlay -->
      <Transition name="slide-panel">
        <UserManual v-if="isManualOpen" @close="isManualOpen = false" />
      </Transition>
    </main>
  </div>

  <Footer />
</template>

<style scoped>
/* ===== SHELL LAYOUT ===== */
.app-shell {
  display: flex;
  flex: 1;
  height: calc(100vh - 52px);
  overflow: hidden;
  background: #0a0a0f;
}

/* ===== SIDEBAR ===== */
.sidebar {
  width: 330px;
  min-width: 330px;
  background: #0d0d14;
  border-right: 1px solid #1a2535;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: #1a2535 transparent;
}

.sidebar::-webkit-scrollbar { width: 4px; }
.sidebar::-webkit-scrollbar-thumb { background: #1a2535; border-radius: 2px; }

.sidebar-section {
  padding: 12px 14px 10px;
}

.sidebar-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #2d4a6b;
  margin-bottom: 8px;
}

.sidebar-divider {
  height: 1px;
  background: #1a2535;
  margin: 0 14px;
}

.label-lock {
  font-size: 10px;
  margin-left: 4px;
  opacity: 0.6;
}

/* Quick access */

.qa-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 8px;
  border: none;
  background: transparent;
  color: #4a6a8a;
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  cursor: pointer;
  border-radius: 3px;
  transition: all 0.15s ease;
  text-align: left;
}

.qa-item:hover {
  background: rgba(0, 229, 255, 0.05);
  color: #00e5ff;
}

.qa-item.active {
  background: rgba(0, 229, 255, 0.08);
  color: #00e5ff;
}

.qa-icon {
  display: flex;
  align-items: center;
  opacity: 0.7;
}

/* ===== MAIN PANEL ===== */
.main-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  min-height: 0;
  background: #0d0f18;
}

/* Tab bar */
.tab-bar {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 16px;
  background: #0a0c14;
  border-bottom: 1px solid #1a2535;
  flex-shrink: 0;
}

/* ===== STRING SELECTOR BAR ===== */
.str-selector-bar {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 7px 16px;
  background: #080a12;
  border-bottom: 1px solid #1a2535;
  flex-shrink: 0;
  overflow-x: auto;
  scrollbar-width: none;
}
.str-selector-bar::-webkit-scrollbar { display: none; }

.str-sel-label {
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #2d4a6b;
  white-space: nowrap;
  flex-shrink: 0;
  margin-right: 2px;
}

.str-sel-sep {
  width: 1px;
  height: 16px;
  background: #1a2535;
  flex-shrink: 0;
  margin: 0 2px;
}

.str-pill {
  display: inline-flex;
  align-items: center;
  gap: 5px;
  padding: 4px 10px;
  border-radius: 20px;
  border: 1px solid #1e2d3d;
  background: transparent;
  color: #4a6a8a;
  font-family: 'Space Mono', monospace;
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.06em;
  cursor: pointer;
  white-space: nowrap;
  flex-shrink: 0;
  transition: all 0.18s ease;
}

.str-pill:hover:not(:disabled) {
  border-color: #2d4a6b;
  color: #c8d0e0;
  background: rgba(255,255,255,0.03);
}

.str-pill:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.str-pill.sp-active {
  border-color: #00e5ff;
  color: #00e5ff;
  background: rgba(0, 229, 255, 0.08);
  box-shadow: 0 0 8px rgba(0, 229, 255, 0.15);
}

.str-pill.sp-valid   { border-color: rgba(16,185,129,0.4); color: #10b981; }
.str-pill.sp-valid.sp-active   { background: rgba(16,185,129,0.1); box-shadow: 0 0 8px rgba(16,185,129,0.2); }
.str-pill.sp-invalid { border-color: rgba(239,68,68,0.4);  color: #ef4444; }
.str-pill.sp-invalid.sp-active { background: rgba(239,68,68,0.1); box-shadow: 0 0 8px rgba(239,68,68,0.2); }
.str-pill.sp-running { border-color: rgba(0,229,255,0.5); color: #00e5ff; }

/* Live dot */
.sp-live-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: #2d4a6b;
  display: inline-block;
  flex-shrink: 0;
  transition: background 0.3s;
}
.sp-live-dot--anim {
  background: #00e5ff;
  animation: live-pulse 1.1s ease-in-out infinite;
}
@keyframes live-pulse {
  0%, 100% { opacity: 1; box-shadow: 0 0 0 0 rgba(0,229,255,0.5); }
  50%       { opacity: 0.6; box-shadow: 0 0 0 4px rgba(0,229,255,0); }
}

/* Numbered string pills */
.sp-numbered { gap: 4px; }

.sp-num {
  font-size: 9px;
  color: inherit;
  opacity: 0.7;
  flex-shrink: 0;
}
.sp-str {
  font-size: 10px;
  max-width: 80px;
  overflow: hidden;
  text-overflow: ellipsis;
}
.sp-dot {
  font-size: 9px;
  flex-shrink: 0;
  font-weight: 900;
}
.sp-dot--ok   { color: #10b981; }
.sp-dot--fail { color: #ef4444; }
.sp-dot--spin {
  color: #00e5ff;
  display: inline-block;
  animation: spin 1s linear infinite;
}
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

/* Visualization canvas */
.viz-canvas {
  flex: 1;
  overflow: hidden;
  position: relative;
  display: flex;
  flex-direction: column;
  min-height: 0;
  /* Dot grid background */
  background-color: #0d0f18;
  background-image: radial-gradient(circle, #1e2d3d 1px, transparent 1px);
  background-size: 24px 24px;
}

.viz-inner {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

/* User Manual Slide Transition */
.slide-panel-enter-active,
.slide-panel-leave-active {
  transition: transform 0.3s cubic-bezier(0.2, 0.8, 0.2, 1), opacity 0.3s ease;
}
.slide-panel-enter-from,
.slide-panel-leave-to {
  transform: translateX(100%);
  opacity: 0;
}

/* Empty state */
.empty-canvas {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #4a6a8a;
  gap: 16px;
  padding-bottom: 40px;
}

.empty-icon {
  opacity: 0.4;
}

.terminal-header-sidebar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.terminal-running-badge {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  color: #00e5ff;
  background: rgba(0, 229, 255, 0.1);
  border: 1px solid rgba(0, 229, 255, 0.25);
  padding: 3px 8px;
  border-radius: 4px;
  animation: pulse-badge 1.5s infinite;
  text-transform: uppercase;
}

@keyframes pulse-badge {
  0%, 100% { opacity: 1; border-color: rgba(0, 229, 255, 0.4); }
  50%      { opacity: 0.6; border-color: rgba(0, 229, 255, 0.1); }
}

/* ===== RESPONSIVE ===== */
@media (max-width: 768px) {
  .app-shell {
    flex-direction: column;
    height: auto;
    overflow: auto;
  }

  .sidebar {
    width: 100%;
    min-width: unset;
    border-right: none;
    border-bottom: 1px solid #1a2535;
    max-height: 280px;
  }

  .main-panel {
    height: auto;
    min-height: 60vh;
  }

  .viz-canvas {
    min-height: 350px;
  }
}
</style>
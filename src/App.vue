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
const simulationInputs = ref([])

const automata = ref('automata-theory project')

const problems = ref([
  {
    id: 1,
    label: '(bab+bbb) b* a* (a*+b*) (ab)* (aba) (bab+aba)* bb (a+b)* (bab+aba) (a+b)*',
    regexStr: '(bab+bbb) b* a* (a*+b*) (ab)* (aba) (bab+aba)* bb (a+b)* (bab+aba) (a+b)*'
  },
  {
    id: 2,
    label: '(1+0)* 0* 1* (111+00+101) (1+0)* (101+01+000) (1+0)* (101+000)*',
    regexStr: '(1+0)* 0* 1* (111+00+101) (1+0)* (101+01+000) (1+0)* (101+000)*'
  }
])

const selectedProblemIndex = ref(-1)
const currentRegex = computed(() => {
    if (selectedProblemIndex.value === -1) return ''
    return problems.value[selectedProblemIndex.value]?.regexStr || ''
})

const updateSimulationInputs = (newInputs) => {
  simulationInputs.value = newInputs
}

const currentTestString = ref('')
const simulationKey = ref(0)
const runSimulation = (str) => {
  currentTestString.value = str
  simulationKey.value++
}
</script>

<template>
  <NavBar @open-manual="activeAutomata = 'manual'; selectedProblemIndex = -1" />

  <div class="app-shell">
    <!-- ===== LEFT SIDEBAR ===== -->
    <aside class="sidebar">
      <div class="sidebar-section">
        <div class="sidebar-label">Active Problem</div>
        <div class="active-problem-card" :class="{ 'has-selection': selectedProblemIndex !== -1 }">
          <div class="problem-id-tag">
            {{ selectedProblemIndex !== -1 ? `PROBLEM_${problems[selectedProblemIndex].id}` : 'NONE_SELECTED' }}
          </div>
          <div class="problem-type-tag">
            {{ selectedProblemIndex !== -1 ? activeAutomata.toUpperCase() + ' Implementation' : 'Select a problem below' }}
          </div>
        </div>
      </div>

      <div class="sidebar-divider"></div>

      <div class="sidebar-section">
        <div class="sidebar-label">Regex Selection</div>
        <Problems
          :problems="problems"
          v-model="selectedProblemIndex"
        />
      </div>

      <div class="sidebar-divider"></div>

      <template v-if="selectedProblemIndex !== -1">
        <div class="sidebar-section">
          <div class="sidebar-label">Quick Access</div>
          <div class="quick-access-list">
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
        </div>
        <div class="sidebar-divider"></div>
      </template>
    </aside>

    <!-- ===== MAIN AREA ===== -->
    <main class="main-panel">

      <!-- Tabs bar -->
      <div class="tab-bar">
        <button
          class="tab-btn"
          :class="{ active: activeAutomata === 'dfa' }"
          @click="activeAutomata = 'dfa'"
        >DFA</button>
        <button
          class="tab-btn"
          :class="{ active: activeAutomata === 'cfg' }"
          @click="activeAutomata = 'cfg'"
        >CFG</button>
        <button
          class="tab-btn"
          :class="{ active: activeAutomata === 'pda' }"
          @click="activeAutomata = 'pda'"
        >PDA</button>
      </div>

      <!-- Visualization canvas -->
      <div class="viz-canvas">
        <div v-if="activeAutomata === 'manual'" class="viz-inner">
          <UserManual />
        </div>
        <template v-else-if="selectedProblemIndex !== -1">
          <div v-if="activeAutomata === 'dfa'" class="viz-inner">
            <Diagram
              :problemId="problems[selectedProblemIndex].id"
              :testString="currentTestString"
              :simKey="simulationKey"
            />
          </div>
          <div v-else-if="activeAutomata === 'cfg'" class="viz-inner">
            <CFGVisualization
              :problemId="problems[selectedProblemIndex].id"
              :testString="currentTestString"
            />
          </div>
          <div v-else-if="activeAutomata === 'pda'" class="viz-inner">
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

      <!-- Bottom panel: Test Terminal -->
      <div class="bottom-panel" v-if="selectedProblemIndex !== -1">
        <div class="test-terminal-section">
          <div class="terminal-header">
            <span class="terminal-label">Test Terminal</span>
          </div>
          <InputArea
            :regexStr="currentRegex"
            @inputs-updated="updateSimulationInputs"
            @simulate-string="runSimulation"
          />
        </div>
      </div>

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
  width: 220px;
  min-width: 220px;
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
  padding: 16px 14px 12px;
}

.sidebar-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #2d4a6b;
  margin-bottom: 10px;
}

.sidebar-divider {
  height: 1px;
  background: #1a2535;
  margin: 0 14px;
}

/* Active problem card */
.active-problem-card {
  background: #111827;
  border: 1px solid #1e2d3d;
  border-radius: 4px;
  padding: 10px 12px;
  transition: border-color 0.2s;
}

.active-problem-card.has-selection {
  border-color: #00e5ff;
  box-shadow: 0 0 12px rgba(0, 229, 255, 0.1);
}

.problem-id-tag {
  font-size: 12px;
  font-weight: 700;
  color: #00e5ff;
  letter-spacing: 0.06em;
  margin-bottom: 4px;
}

.problem-type-tag {
  font-size: 10px;
  color: #4a6a8a;
  letter-spacing: 0.04em;
}

/* Quick access */
.quick-access-list {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.qa-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 10px;
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
  width: 100%;
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
  background: #0d0f18;
}

/* Tab bar */
.tab-bar {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 10px 16px;
  background: #0a0c14;
  border-bottom: 1px solid #1a2535;
  flex-shrink: 0;
}

.tab-btn {
  padding: 5px 16px;
  border-radius: 4px;
  border: 1px solid #1e2d3d;
  background: transparent;
  color: #4a6a8a;
  font-family: 'Space Mono', monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.08em;
  cursor: pointer;
  transition: all 0.15s ease;
}

.tab-btn:hover {
  border-color: #2d4a6b;
  color: #c8d0e0;
}

.tab-btn.active {
  background: #00e5ff;
  border-color: #00e5ff;
  color: #0a0a0f;
  box-shadow: 0 0 12px rgba(0, 229, 255, 0.4);
}

/* Visualization canvas */
.viz-canvas {
  flex: 1;
  overflow: auto;
  position: relative;
  /* Dot grid background */
  background-color: #0d0f18;
  background-image: radial-gradient(circle, #1e2d3d 1px, transparent 1px);
  background-size: 24px 24px;
}

.viz-inner {
  min-height: 100%;
  display: flex;
  flex-direction: column;
}

/* Empty state */
.empty-canvas {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.empty-text {
  font-size: 12px;
  letter-spacing: 0.08em;
  color: #1e3a5f;
  text-transform: uppercase;
}

/* Bottom panel */
.bottom-panel {
  border-top: 1px solid #1a2535;
  background: #0a0c14;
  padding: 12px 16px;
  flex-shrink: 0;
}

.terminal-header {
  margin-bottom: 8px;
}

.terminal-label {
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: #2d4a6b;
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
<script setup>
import { ref, watch, onMounted, computed } from 'vue'

const props = defineProps({
    problemId: { type: Number, required: true },
    testString: { type: String, default: '' }
})

const CFG_DATA = {
    1: {
        startSymbol: 'S',
        productions: [
            { lhs: 'S', alts: ['A B C D E (aba) H (bb) G F G'] },
            { lhs: 'A', alts: ['bab', 'bbb'] },
            { lhs: 'B', alts: ['bB', 'λ'] },
            { lhs: 'C', alts: ['aC', 'λ'] },
            { lhs: 'D', alts: ['C', 'B'] },
            { lhs: 'E', alts: ['abE', 'λ'] },
            { lhs: 'F', alts: ['bab', 'aba'] },
            { lhs: 'G', alts: ['aG', 'bG', 'λ'] },
            { lhs: 'H', alts: ['babH', 'abaH', 'λ'] },
        ],
        terminals: ['a', 'b', 'λ', '(', ')'],
        nonTerminals: ['S', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H']
    },
    2: {
        startSymbol: 'S',
        productions: [
            { lhs: 'S', alts: ['A B C D A E A F'] },
            { lhs: 'A', alts: ['1A', '0A', 'λ'] },
            { lhs: 'B', alts: ['0B', 'λ'] },
            { lhs: 'C', alts: ['1C', 'λ'] },
            { lhs: 'D', alts: ['111', '00', '101'] },
            { lhs: 'E', alts: ['101', '01', '000'] },
            { lhs: 'F', alts: ['101F', '000F', 'λ'] },
        ],
        terminals: ['0', '1', 'λ'],
        nonTerminals: ['S', 'A', 'B', 'C', 'D', 'E', 'F']
    }
}

const REGEX_MAP = {
  1: '(bab+bbb) b* a* (a*+b*) (ab)* (aba) (bab+aba)* bb (a+b)* (bab+aba) (a+b)*',
  2: '(1+0)* 0* 1* (111+00+101) (1+0)* (101+01+000) (1+0)* (101+000)*'
}

const cfg = computed(() => CFG_DATA[props.problemId] || CFG_DATA[1])
const problemRegex = computed(() => REGEX_MAP[props.problemId])
const hoveredRow = ref(null)

const tokenizeAlt = (alt) => {
    const tokens = []
    let i = 0
    while (i < alt.length) {
        if (alt[i] === '(' || alt[i] === ')') {
            // Ignore grouping parens in the grammar string representation
            i++
        } else if (alt[i] !== ' ') {
            tokens.push({ text: alt[i], isNT: cfg.value.nonTerminals.includes(alt[i]) })
            i++
        } else {
            i++
        }
    }
    return tokens
}

// DFS Backtracking for Leftmost Derivation
const derivationSteps = computed(() => {
    if (!props.testString) return []
    
    let bestResult = null
    let longestPrefixLength = -1
    let bestFailedTrace = []
    
    const MAX_STEPS = 5000
    let iterations = 0
    
    const dfs = (currentTokens, steps) => {
        if (bestResult) return
        if (iterations++ > MAX_STEPS) return
        
        const ntIdx = currentTokens.findIndex(t => t.isNT)
        
        // If no NTs left, check if exact match
        if (ntIdx === -1) {
            const derivedStr = currentTokens.map(t => t.text).join('').replace(/λ/g, '')
            if (derivedStr === props.testString) {
                bestResult = [...steps, {
                    step: steps.length,
                    stringTokens: currentTokens.map(t => ({ text: t.text, isExpandedNT: false })),
                    rule: 'Valid',
                    isFinal: true
                }]
            } else {
                if (derivedStr.length > longestPrefixLength) {
                    longestPrefixLength = derivedStr.length
                    bestFailedTrace = [...steps]
                }
            }
            return
        }
        
        // Pruning based on prefix
        let prefixStr = ''
        for (let i = 0; i < ntIdx; i++) {
            if (currentTokens[i].text !== 'λ') {
                prefixStr += currentTokens[i].text
            }
        }
        
        // Track the furthest valid path before we prune
        if (prefixStr.length > longestPrefixLength || (prefixStr.length === longestPrefixLength && steps.length > bestFailedTrace.length)) {
            longestPrefixLength = prefixStr.length
            bestFailedTrace = [...steps]
        }
        
        if (!props.testString.startsWith(prefixStr)) {
            return
        }
        
        const nt = currentTokens[ntIdx].text
        const prod = cfg.value.productions.find(p => p.lhs === nt)
        if (!prod) return
        
        const displayTokens = currentTokens.map((t, idx) => ({
            text: t.text,
            isExpandedNT: idx === ntIdx
        }))
        
        for (const alt of prod.alts) {
            let replacement = tokenizeAlt(alt)
            if (alt === 'λ') replacement = [{ text: 'λ', isNT: false }]
            
            const nextTokens = [...currentTokens]
            nextTokens.splice(ntIdx, 1, ...replacement)
            
            const nextSteps = [...steps, {
                step: steps.length,
                stringTokens: displayTokens,
                rule: `${nt} → ${alt}`
            }]
            
            dfs(nextTokens, nextSteps)
        }
    }
    
    const initialTokens = [{ text: cfg.value.startSymbol, isNT: true }]
    dfs(initialTokens, [])
    
    if (!bestResult) {
        if (bestFailedTrace.length > 0) {
            const failedTrace = [...bestFailedTrace]
            // Add a failure indicator step
            failedTrace.push({
                step: failedTrace.length,
                stringTokens: [{ text: 'Mismatch Detected', isExpandedNT: false }],
                rule: 'Invalid',
                isInvalid: true
            })
            return failedTrace
        }
        return [{ step: '-', stringTokens: [{ text: 'Invalid String', isExpandedNT: false }], rule: 'Fail', isInvalid: true }]
    }
    
    return bestResult
})

</script>

<template>
  <div class="cfg-wrap">
    <!-- Header -->
    <div class="cfg-header">
      <div class="header-left">
        <span class="badge">CFG</span>
        <span class="title">Problem {{ problemId }}</span>
      </div>
      <div class="header-right">
        <span class="leg-item"><span class="dot nt-dot"></span>Non-terminal</span>
        <span class="leg-item"><span class="dot t-dot"></span>Terminal</span>
      </div>
    </div>

    <!-- Regex -->
    <div class="regex-wrap" v-if="problemRegex">
      <span class="regex-label">Regex</span>
      <code class="regex-code">{{ problemRegex }}</code>
    </div>

    <div class="cfg-content">
      <!-- Productions List -->
      <div class="rule-card">
        <div class="rule-card-head">
          <span>Productions</span>
          <span class="rule-count">{{ cfg.productions.length }} rules</span>
        </div>
        <div class="rule-list">
          <div
            v-for="(prod, idx) in cfg.productions"
            :key="idx"
            class="rule-row"
            :class="{ hovered: hoveredRow === idx }"
            @mouseenter="hoveredRow = idx"
            @mouseleave="hoveredRow = null"
          >
            <span class="lhs">{{ prod.lhs }}</span>
            <span class="arrow">→</span>
            <span class="rhs-group">
              <template v-for="(alt, ai) in prod.alts" :key="ai">
                <span v-if="ai > 0" class="pipe">|</span>
                <span
                  v-for="(tok, ti) in tokenizeAlt(alt)"
                  :key="ti"
                  :class="['tok', tok.isNT ? 'nt' : 't']"
                >{{ tok.text }}</span>
              </template>
            </span>
            <span class="row-num">{{ idx + 1 }}</span>
          </div>
        </div>
      </div>

      <!-- Simulation Trace -->
      <div class="sim-card">
        <div class="rule-card-head trace-head">
          <span style="display:flex; align-items:center; gap:8px;">
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#00e5ff" stroke-width="2"><path d="M22 12h-4l-3 9L9 3l-3 9H2"></path></svg>
            DERIVATION TRACE
          </span>
        </div>
        
        <div class="trace-container" v-if="testString">
          <div class="trace-list">
            <div 
              v-for="step in derivationSteps" 
              :key="step.step" 
              :class="['trace-row', step.isFinal ? 'valid-final' : '', step.isInvalid ? 'invalid-final' : '']"
            >
              <div class="step-num">Step {{ step.step }}</div>
              
              <div class="trace-string">
                <template v-for="(tok, idx) in step.stringTokens" :key="idx">
                  <template v-if="tok.text !== 'λ' || step.stringTokens.length === 1">
                    <span v-if="tok.isExpandedNT" class="highlight-nt">(<span class="cyan-txt">{{ tok.text }}</span>)</span>
                    <span v-else class="dim-txt">{{ tok.text }}</span>
                  </template>
                </template>
              </div>

              <div class="step-icon" v-if="step.isFinal && step.rule === 'Valid'">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path><polyline points="22 4 12 14.01 9 11.01"></polyline></svg>
              </div>
              <div class="step-icon invalid-icon" v-if="step.isInvalid">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"></circle><line x1="15" y1="9" x2="9" y2="15"></line><line x1="9" y1="9" x2="15" y2="15"></line></svg>
              </div>
            </div>
          </div>
        </div>
        <div class="empty-state" v-else>
          Enter a test string in the Test Terminal.
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* ========== AUTOMATALAB DARK TERMINAL UI ========== */
.cfg-wrap {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
  height: 100%;
  padding: 16px;
  background: #0a0a0f;
  color: #c8d0e0;
  font-family: 'Space Mono', monospace;
  overflow-y: auto;
}

/* Header */
.cfg-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-bottom: 12px;
  border-bottom: 1px solid #1e2d3d;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.badge {
  background: rgba(0, 229, 255, 0.1);
  color: #00e5ff;
  border: 1px solid rgba(0, 229, 255, 0.3);
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.1em;
}

.title {
  font-family: 'Rajdhani', sans-serif;
  font-size: 20px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.05em;
  text-transform: uppercase;
}

.header-right {
  display: flex;
  gap: 16px;
}

.leg-item {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 11px;
  color: #8b9bb4;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.dot {
  width: 8px;
  height: 8px;
  border-radius: 2px;
}

.nt-dot { background: #f0a500; box-shadow: 0 0 8px rgba(240, 165, 0, 0.4); }
.t-dot  { background: #00e5ff; box-shadow: 0 0 8px rgba(0, 229, 255, 0.4); }

/* Regex */
.regex-wrap {
  display: flex;
  flex-direction: column;
  gap: 8px;
  background: #111827;
  border: 1px solid #1e2d3d;
  border-radius: 4px;
  padding: 12px 16px;
}

.regex-label {
  font-size: 10px;
  color: #00e5ff;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.regex-code {
  color: #c8d0e0;
  font-size: 13px;
  word-break: break-all;
  line-height: 1.5;
}

/* Content Layout */
.cfg-content {
  display: grid;
  grid-template-columns: 1fr 1.5fr;
  gap: 16px;
  align-items: stretch;
  flex: 1;
  min-height: 0;
}

@media (max-width: 1000px) {
  .cfg-content {
    grid-template-columns: 1fr;
  }
}

/* Rule Card & Table Card */
.rule-card, .sim-card {
  background: #111827;
  border: 1px solid #1e2d3d;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.rule-card-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 16px;
  background: rgba(30, 45, 61, 0.4);
  border-bottom: 1px solid #1e2d3d;
  font-size: 11px;
  font-weight: 700;
  color: #8b9bb4;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

.rule-count {
  color: #00e5ff;
  background: rgba(0, 229, 255, 0.1);
  padding: 2px 6px;
  border-radius: 3px;
}

/* Productions List */
.rule-list {
  display: flex;
  flex-direction: column;
  flex: 1;
  min-height: 0;
  overflow-y: auto;
}

.rule-row {
  display: flex;
  align-items: flex-start;
  padding: 6px 12px;
  border-bottom: 1px solid rgba(30, 45, 61, 0.3);
  font-size: 12px;
  transition: background 0.2s;
  position: relative;
}

.rule-row:last-child {
  border-bottom: none;
}

.rule-row.hovered {
  background: rgba(30, 45, 61, 0.4);
}

.lhs {
  font-weight: 700;
  color: #f0a500;
  min-width: 20px;
}

.arrow {
  color: #4a6a8a;
  margin: 0 12px;
}

.rhs-group {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 4px;
  flex: 1;
}

.pipe {
  color: #4a6a8a;
  margin: 0 6px;
}

.tok {
  padding: 0 2px;
}
.tok.nt { color: #f0a500; font-weight: 700; }
.tok.t  { color: #00e5ff; }

.row-num {
  font-size: 10px;
  color: #4a6a8a;
  min-width: 20px;
  text-align: right;
  margin-left: 12px;
}

.rule-row.hovered .row-num {
  color: #00e5ff;
}

.rule-card-head.trace-head {
  color: #00e5ff;
  font-family: 'Rajdhani', sans-serif;
  font-size: 13px;
  letter-spacing: 0.15em;
  border-bottom: 1px solid #1e2d3d;
}

/* Trace List */
.trace-container {
  max-height: 400px;
  overflow-y: auto;
  background: #111827;
  padding: 12px;
}

.trace-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.trace-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(30, 45, 61, 0.4);
  border: 1px solid rgba(30, 45, 61, 0.8);
  border-radius: 6px;
  padding: 14px 16px;
  position: relative;
}

.step-num {
  font-size: 10px;
  color: #4a6a8a;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  font-weight: 700;
  width: 60px;
}

.trace-string {
  flex: 1;
  text-align: center;
  font-size: 14px;
  letter-spacing: 0.05em;
  padding: 0 16px;
}

.dim-txt {
  color: #6b7a90;
}

.highlight-nt {
  color: #4a6a8a;
  font-weight: 700;
}

.cyan-txt {
  color: #00e5ff;
}

/* Valid Final Step */
.trace-row.valid-final {
  background: rgba(16, 185, 129, 0.15);
  border-color: #10b981;
}

.trace-row.valid-final .step-num,
.trace-row.valid-final .dim-txt {
  color: #10b981;
}

.step-icon {
  color: #10b981;
  display: flex;
  align-items: center;
}

/* Invalid Final Step */
.trace-row.invalid-final {
  background: rgba(239, 68, 68, 0.15);
  border-color: #ef4444;
}

.trace-row.invalid-final .step-num,
.trace-row.invalid-final .dim-txt,
.trace-row.invalid-final .highlight-nt,
.trace-row.invalid-final .cyan-txt {
  color: #ef4444;
}

.invalid-icon {
  color: #ef4444;
}

.empty-state {
  padding: 40px 20px;
  text-align: center;
  color: #4a6a8a;
  font-size: 12px;
  font-style: italic;
}

/* Custom Scrollbar */
::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}
::-webkit-scrollbar-track {
  background: #0a0a0f;
}
::-webkit-scrollbar-thumb {
  background: #1e2d3d;
  border-radius: 3px;
}
::-webkit-scrollbar-thumb:hover {
  background: #4a6a8a;
}
</style>
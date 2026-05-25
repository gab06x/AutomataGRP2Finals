<script setup>
import Regex1 from '../assets/Regex1.jpg'
import Regex2 from '@/assets/Regex2.png'

defineProps({
    problemId: { type: Number, required: true }
})

const REGEX_MAP = {
  1: '(bab+bbb) b* a* (a*+b*) (ab)* (aba) (bab+aba)* bb (a+b)* (bab+aba) (a+b)*',
  2: '(1+0)* 0* 1* (111+00+101) (1+0)* (101+01+000) (1+0)* (101+000)*'
}
</script>

<template>
  <div class="pda-wrap">
    
    <!-- Header -->
    <div class="pda-header">
      <div class="header-left">
        <span class="badge">PDA</span>
        <span class="title">Problem {{ problemId }}</span>
      </div>
      <div class="header-right">
        <span class="attribution">Assigned to Luis (Regex 1) & Marc (Regex 2)</span>
      </div>
    </div>

    <!-- Regex -->
    <div class="regex-wrap" v-if="REGEX_MAP[problemId]">
      <span class="regex-label">Regex</span>
      <code class="regex-code">{{ REGEX_MAP[problemId] }}</code>
    </div>

    <!-- Image Container -->
    <div class="pda-viz-card">
      <div class="viz-head">
        <span style="display:flex; align-items:center; gap:8px;">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#00e5ff" stroke-width="2"><rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect><circle cx="8.5" cy="8.5" r="1.5"></circle><polyline points="21 15 16 10 5 21"></polyline></svg>
          PUSHDOWN AUTOMATA DIAGRAM
        </span>
      </div>
      <div class="image-viewport">
        <div v-if="problemId === 1">
          <img :src="Regex1" alt="PDA for Regex 1" class="pda-image" />
        </div>
        
        <div v-else-if="problemId === 2">
          <img :src="Regex2" alt="PDA for Regex 2" class="pda-image" />
        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
/* ========== AUTOMATALAB DARK TERMINAL UI ========== */
.pda-wrap {
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
.pda-header {
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
  align-items: center;
}

.attribution {
  font-size: 11px;
  color: #4a6a8a;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

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

/* Viz Card */
.pda-viz-card {
  background: #111827;
  border: 1px solid #1e2d3d;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.viz-head {
  padding: 10px 16px;
  background: rgba(30, 45, 61, 0.4);
  border-bottom: 1px solid #1e2d3d;
  color: #00e5ff;
  font-family: 'Rajdhani', sans-serif;
  font-size: 13px;
  letter-spacing: 0.15em;
  font-weight: 700;
}

.image-viewport {
  padding: 24px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #0f141e; /* Deep dark background */
  min-height: 300px;
}

.pda-image {
  max-width: 100%;
  height: auto;
  border-radius: 6px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
  border: 1px solid #1e2d3d;
  background: #fff; /* White background in case the image is transparent, so text is readable */
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

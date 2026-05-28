<script setup>
import { ref } from 'vue'
import Regex1 from '../assets/Regex1.jpg'
import Regex2 from '@/assets/Regex2.png'

const isFullscreen = ref(false)

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
    
    <!-- Header Removed to save space -->

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
        <span class="attribution">Assigned to Luis (Regex 1) & Marc (Regex 2)</span>
      </div>
      <div class="image-viewport">
        <img v-if="problemId === 1" :src="Regex1" alt="PDA for Regex 1" class="pda-image" @click="isFullscreen = true" />
        <img v-else-if="problemId === 2" :src="Regex2" alt="PDA for Regex 2" class="pda-image" @click="isFullscreen = true" />
      </div>
    </div>

    <!-- Fullscreen Overlay -->
    <div v-if="isFullscreen" class="fullscreen-overlay" @click="isFullscreen = false">
      <button class="close-fs-btn" @click.stop="isFullscreen = false">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M18 6L6 18M6 6l12 12"/></svg>
      </button>
      <img v-if="problemId === 1" :src="Regex1" class="pda-fs-image" @click.stop />
      <img v-else-if="problemId === 2" :src="Regex2" class="pda-fs-image" @click.stop />
    </div>

  </div>
</template>

<style scoped>
/* ========== AUTOMATALAB DARK TERMINAL UI ========== */
.pda-wrap {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
  height: 100%;
  padding: 8px;
  background: #0a0a0f;
  color: #c8d0e0;
  font-family: 'Space Mono', monospace;
  overflow-y: auto;
}

.attribution {
  font-size: 10px;
  color: #4a6a8a;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

/* Regex */
.regex-wrap {
  display: flex;
  flex-direction: column;
  gap: 4px;
  background: #111827;
  border: 1px solid #1e2d3d;
  border-radius: 4px;
  padding: 8px 12px;
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
  font-size: 11px;
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
  flex: 1;
}

.viz-head {
  padding: 6px 12px;
  background: rgba(30, 45, 61, 0.4);
  border-bottom: 1px solid #1e2d3d;
  color: #00e5ff;
  font-family: 'Rajdhani', sans-serif;
  font-size: 11px;
  letter-spacing: 0.15em;
  font-weight: 700;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.image-viewport {
  padding: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #0f141e; /* Deep dark background */
  min-height: 0;
  flex: 1;
  overflow: hidden;
}

.pda-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  border-radius: 6px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
  border: 1px solid #1e2d3d;
  background: #fff;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.pda-image:hover {
  transform: scale(1.02);
  border-color: #00e5ff;
}

/* Fullscreen Overlay */
.fullscreen-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(10, 10, 15, 0.95);
  backdrop-filter: blur(10px);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
}

.close-fs-btn {
  position: absolute;
  top: 20px;
  right: 20px;
  background: rgba(0, 229, 255, 0.1);
  border: 1px solid rgba(0, 229, 255, 0.3);
  color: #00e5ff;
  border-radius: 50%;
  width: 44px;
  height: 44px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
  transition: all 0.2s;
}

.close-fs-btn:hover {
  background: rgba(255, 68, 68, 0.2);
  color: #ff4444;
  border-color: #ff4444;
}

.pda-fs-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  border-radius: 8px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.8);
  background: #fff;
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

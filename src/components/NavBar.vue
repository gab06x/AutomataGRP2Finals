<script setup>
import { onMounted } from 'vue'

defineEmits(['open-manual'])

onMounted(() => {
  document.body.classList.add('dark-mode')
})
</script>

<template>
  <nav class="navbar">
    <!-- Logo + Title -->
    <div class="nav-brand">
      <div class="nav-logo">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <circle cx="5" cy="12" r="3" stroke="#00e5ff" stroke-width="1.5"/>
          <circle cx="19" cy="5" r="3" stroke="#c8ff00" stroke-width="1.5"/>
          <circle cx="19" cy="19" r="3" stroke="#c8ff00" stroke-width="1.5"/>
          <path d="M8 12h4M13 8l6-3M13 16l6 3" stroke="#00e5ff" stroke-width="1.5" stroke-linecap="round"/>
        </svg>
      </div>
      <span class="nav-title">AutomaTe</span>
      <span class="nav-version">v1.0.0</span>
    </div>

    <!-- Right Actions -->
    <div class="nav-actions">
      <button class="manual-btn" @click="$emit('open-manual')">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"></path><path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"></path></svg>
        USER MANUAL
      </button>
    </div>

    <!-- Scan line decoration -->
    <div class="scan-line"></div>
  </nav>
</template>

<style scoped>
.navbar {
  display: flex;
  align-items: center;
  padding: 0 24px;
  justify-content: space-between;
  height: 52px;
  background: #0d0d14;
  border-bottom: 1px solid #1a2535;
  position: relative;
  overflow: hidden;
  flex-shrink: 0;
  z-index: 100;
}

.navbar::before {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, #00e5ff 30%, #c8ff00 70%, transparent);
  opacity: 0.5;
}

.nav-brand {
  display: flex;
  align-items: center;
  gap: 10px;
}

.nav-logo {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: rgba(0, 229, 255, 0.05);
  border: 1px solid rgba(0, 229, 255, 0.2);
  border-radius: 6px;
}

.nav-title {
  font-family: 'Rajdhani', 'Space Mono', monospace;
  font-size: 18px;
  font-weight: 700;
  letter-spacing: 0.05em;
  color: #e8edf5;
  text-transform: uppercase;
}

.nav-version {
  font-size: 10px;
  color: #2d4a6b;
  background: #111827;
  border: 1px solid #1e2d3d;
  padding: 2px 6px;
  border-radius: 3px;
  letter-spacing: 0.05em;
  margin-left: 2px;
}

.nav-actions {
  display: flex;
  align-items: center;
}

.manual-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(0, 229, 255, 0.1);
  border: 1px solid rgba(0, 229, 255, 0.3);
  color: #00e5ff;
  padding: 6px 14px;
  border-radius: 4px;
  font-family: 'Space Mono', monospace;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.1em;
  cursor: pointer;
  transition: all 0.2s ease;
}

.manual-btn:hover {
  background: rgba(0, 229, 255, 0.2);
  box-shadow: 0 0 10px rgba(0, 229, 255, 0.2);
}

.scan-line {
  position: absolute;
  top: 0;
  left: -100%;
  width: 60%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(0, 229, 255, 0.03), transparent);
  animation: navScan 6s linear infinite;
}

@keyframes navScan {
  from { left: -60%; }
  to { left: 120%; }
}

@media (max-width: 640px) {
  .navbar { padding: 0 14px; height: 48px; }
  .nav-title { font-size: 15px; }
}
</style>
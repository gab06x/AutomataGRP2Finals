<template>
  <footer class="bottom-navbar">
    <div class="footer-content">
      <div class="course-info">
        AUTOMATA THEORY AND FORMAL LANGUAGES &mdash; <strong>DLSUD</strong>
      </div>
      <div class="team-info">
        developed by
        <span>Gabriel Castillo</span>,
        <span>Marc Daniel Cajayon</span>,
        <span>Luis Miguel Trias</span>,
        <span>Gian Paolo Ybut</span>
      </div>
    </div>
  </footer>
</template>

<style scoped>
.bottom-navbar {
  width: 100%;
  background: #0d0d14;
  color: #2d4a6b;
  padding: 12px 24px;
  border-top: 1px solid #1a2535;
  font-family: 'Space Mono', monospace;
  flex-shrink: 0;
  position: relative;
}

.bottom-navbar::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, #1e2d3d 30%, #1e2d3d 70%, transparent);
}

.footer-content {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
}

.course-info {
  font-size: 10px;
  letter-spacing: 0.1em;
  font-weight: 500;
  text-transform: uppercase;
  color: #2d4a6b;
}

.course-info strong {
  color: #00e5ff;
  margin-left: 2px;
  font-weight: 700;
}

.team-info {
  font-size: 10px;
  color: #2d4a6b;
  text-align: right;
  line-height: 1.5;
  letter-spacing: 0.04em;
}

.team-info span {
  color: #4a6a8a;
  font-weight: 500;
}

@media (max-width: 850px) {
  .footer-content {
    flex-direction: column;
    text-align: center;
    gap: 8px;
  }

  .team-info {
    text-align: center;
    word-break: break-word;
    overflow-wrap: break-word;
    max-width: 100%;
  }

  .bottom-navbar {
    padding: 16px;
    box-sizing: border-box;
  }
}
</style>

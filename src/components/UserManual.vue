<script setup>
const emit = defineEmits(['close'])
</script>

<template>
  <div class="manual-plain">
    <div class="manual-header">
      <div class="header-titles">
        <h1 class="manual-title">AutomaTe — User Guide</h1>
        <div class="manual-subtitle">Interactive Computation Theory Visualizer &nbsp;·&nbsp; S-CSPC327</div>
      </div>
      <button class="close-btn" @click="emit('close')" title="Close Manual">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none"><path d="M18 6L6 18M6 6l12 12" stroke="currentColor" stroke-width="2" stroke-linecap="round"/></svg>
      </button>
    </div>

    <div class="manual-content">

      <!-- INTRODUCTION -->
      <section>
        <h2>Introduction</h2>
        <p>
          <strong>AutomaTe</strong> is a web-based platform for exploring computation theory through interactive visualization.
          Select one of two predefined regular expression problems, type up to five test strings, and instantly see how each string
          is processed by a <strong>Deterministic Finite Automaton (DFA)</strong>, parsed by a <strong>Context-Free Grammar (CFG)</strong>,
          and modeled by a <strong>Pushdown Automaton (PDA)</strong>.
        </p>
      </section>

      <!-- LAYOUT -->
      <section>
        <h2>Interface Layout</h2>

        <h3>Top Navbar</h3>
        <ul>
          <li><strong>Regex Selection</strong> — Located at the very top. Click Problem 1 or Problem 2 to load that problem's DFA, CFG, and PDA.</li>
          <li><strong>User Manual</strong> — Slide-out side panel for guidance.</li>
        </ul>

        <h3>Left Sidebar (Test Terminal)</h3>
        <p>The main input area. You can enter up to <strong>5 test strings</strong> simultaneously.</p>
        <ul>
          <li>Each row shows two badges: a <strong>Regex badge</strong> (instant pattern match check) and a <strong>DFA badge</strong> (result after simulation).</li>
          <li><strong>Run</strong> — Simulates a single row's string on the DFA and updates the CFG derivation.</li>
          <li><strong>Run All</strong> — Simulates strings 1 → 5 in sequence, animating each one before advancing.</li>
          <li><strong>Cancel</strong> — Stops a Run All mid-sequence.</li>
          <li><strong>Clear All</strong> — Clears all five inputs and resets all results and stored traces.</li>
        </ul>

        <h3>Main View Tabs</h3>
        <p>Switch between the three visualizer panels at the top of the main canvas:</p>
        <ul>
          <li><strong>State Map (DFA)</strong> — Animated state-machine diagram with tape and accept/reject banner.</li>
          <li><strong>CFG Tree</strong> — Grammar production rules + step-by-step leftmost derivation trace.</li>
          <li><strong>Execution Logs (PDA)</strong> — Structural pushdown automaton diagram.</li>
        </ul>

        <h3>String Selector Bar <span class="badge-inline">View:</span></h3>
        <p>Appears below the tab bar once you have typed at least one test string.</p>
        <ul>
          <li><strong>Numbered pills (1–5)</strong> — Click any pill to instantly view that string's stored DFA result and CFG derivation without re-running the animation. The pill turns <span class="col-green">green ✓</span> or <span class="col-red">red ✗</span> after its simulation completes. Click the same pill again to return to the active terminal string.</li>
          <li>Pills are <em>disabled</em> during an active Run All to prevent conflicts.</li>
        </ul>
      </section>

      <!-- DFA -->
      <section>
        <h2>State Map (DFA)</h2>
        <p>Renders the selected regex as a directed state-transition diagram powered by D3.js.</p>
        <ul>
          <li><strong>Start state (–)</strong> — Amber-outlined node; the machine always begins here.</li>
          <li><strong>Accept state (+)</strong> — Double-bordered, lime-green node; reaching this state means the string is accepted.</li>
          <li><strong>Trap state (T)</strong> — Any unrecoverable rejection path leads here.</li>
          <li><strong>Input Tape</strong> — Characters scroll left-to-right; the active character is highlighted in cyan.</li>
          <li><strong>Trail</strong> — Previously visited nodes and edges glow to show the path taken.</li>
          <li><strong>Result Banner</strong> — "✓ String Accepted" (lime) or "✕ String Rejected" (red) appears when the animation finishes.</li>
        </ul>
        <p>
          When you click a numbered pill in the <em>View</em> bar, the DFA immediately shows the <strong>final highlighted state</strong>
          of that string's stored trace — no waiting for re-animation.
        </p>
      </section>

      <!-- CFG -->
      <section>
        <h2>CFG Derivation</h2>
        <p>Displays the Context-Free Grammar for the active problem and performs a leftmost derivation trace.</p>

        <h3>Productions Panel</h3>
        <ul>
          <li>Lists every production rule. <span class="col-amber">Amber tokens</span> are non-terminals; <span class="col-cyan">cyan tokens</span> are terminals.</li>
          <li>Hover over a row to highlight it.</li>
        </ul>

        <h3>Derivation Trace Panel</h3>
        <ul>
          <li>Shows each expansion step of the leftmost derivation for the selected string.</li>
          <li>The currently expanding non-terminal is highlighted in cyan; all others are dimmed.</li>
          <li>A <span class="col-green">green final row</span> means the string was derived successfully.</li>
          <li>A <span class="col-red">red row</span> means the string cannot be generated by the grammar.</li>
        </ul>
        <p>
          The CFG trace updates whenever you change the active string via the <em>View</em> selector bar,
          even while a DFA simulation is running on a different string.
        </p>
      </section>

      <!-- PDA -->
      <section>
        <h2>Pushdown Automaton (PDA)</h2>
        <p>
          Shows the structural diagram of the PDA corresponding to the active problem.
          This is a <strong>static reference model</strong> illustrating the stack transitions (push / pop / read)
          that back up the DFA and CFG.
        </p>
        <ul>
          <li><strong>Fullscreen Preview</strong> — Click directly on the PDA image to open a large, fully responsive fullscreen overlay for detailed inspection.</li>
        </ul>
      </section>

      <!-- PROBLEMS -->
      <section>
        <h2>Regex Problems</h2>

        <h3>Problem 1</h3>
        <p class="regex-block">(bab + bbb) b* a* (a* + b*) (ab)* (aba) (bab + aba)* bb (a + b)* (bab + aba) (a + b)*</p>
        <p>Accepts strings over {a, b} that match all segments in order: a required prefix of <em>bab</em> or <em>bbb</em>, optional <em>b</em>-runs and <em>a</em>-runs, an optional interleaved section, a fixed <em>aba</em> landmark, an optional repeating group, a mandatory <em>bb</em>, a free suffix, a fixed <em>bab</em> or <em>aba</em>, and a free tail.</p>

        <h3>Problem 2</h3>
        <p class="regex-block">(1 + 0)* 0* 1* (111 + 00 + 101) (1 + 0)* (101 + 01 + 000) (1 + 0)* (101 + 000)*</p>
        <p>Accepts binary strings over {0, 1} containing specific fixed sub-patterns (<em>111</em>, <em>00</em>, or <em>101</em>) and (<em>101</em>, <em>01</em>, or <em>000</em>) in sequence, with optional free bits between and around them, plus an optional repeating tail.</p>
      </section>

      <!-- TROUBLESHOOTING -->
      <section>
        <h2>Tips &amp; Troubleshooting</h2>
        <ul>
          <li><strong>Regex badge vs. DFA badge:</strong> The <em>Regex badge</em> is instant (JavaScript regex check). The <em>DFA badge</em> only appears after you run the simulation for that row.</li>
          <li><strong>Run All skips empty rows</strong> — only non-empty strings trigger an animation step.</li>
          <li><strong>Viewing stored results:</strong> Click a numbered pill after simulations are done to compare strings side-by-side without re-running anything.</li>
          <li><strong>CFG on large strings:</strong> The derivation engine uses DFS with a 5 000-step cap. Very long strings may hit this limit and show a partial trace.</li>
          <li><strong>Best viewed on desktop</strong> — The DFA diagram is optimized for wider screens. Mobile layout stacks panels vertically.</li>
        </ul>
      </section>

      <!-- PROJECT DETAILS -->
      <section class="project-details">
        <h2>Project Details</h2>
        <p><strong>Course:</strong> S-CSPC327 — Theory of Automata and Formal Languages</p>
        <p><strong>Technology Stack:</strong> Vue.js 3 · D3.js · Vite · Vanilla CSS</p>
        <p><strong>Development Team:</strong> Gabriel Castillo (Lead Developer), Marc Daniel Cajayon, Luis Miguel Trias, Gian Paolo Ybut</p>
        <div class="footer-credit">
          AUTOMATA THEORY AND FORMAL LANGUAGES — DLSUD<br>
          developed by Gabriel Castillo · Marc Daniel Cajayon · Luis Miguel Trias · Gian Paolo Ybut
        </div>
      </section>

    </div>
  </div>
</template>

<style scoped>
.manual-plain {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  width: 500px;
  max-width: 100%;
  padding: 32px 40px;
  background: rgba(10, 10, 15, 0.95);
  backdrop-filter: blur(10px);
  border-left: 1px solid #1e2d3d;
  color: #c8d0e0;
  font-family: 'Space Mono', monospace;
  overflow-y: auto;
  z-index: 100;
  box-shadow: -10px 0 30px rgba(0,0,0,0.5);
}

.manual-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  border-bottom: 1px solid #1e2d3d;
  padding-bottom: 24px;
  margin-bottom: 32px;
}

.close-btn {
  background: transparent;
  border: none;
  color: #4a6a8a;
  cursor: pointer;
  padding: 4px;
  transition: all 0.2s;
}
.close-btn:hover {
  color: #ff4444;
  transform: scale(1.1);
}

.manual-title {
  font-family: 'Rajdhani', sans-serif;
  font-size: 32px;
  font-weight: 700;
  color: #00e5ff;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  margin: 0 0 8px 0;
}

.manual-subtitle {
  font-size: 13px;
  color: #8b9bb4;
  letter-spacing: 0.05em;
}

.manual-content {
  max-width: 820px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

section {
  background: #111827;
  border: 1px solid #1e2d3d;
  border-radius: 6px;
  padding: 24px;
}

h2 {
  font-family: 'Rajdhani', sans-serif;
  font-size: 20px;
  font-weight: 700;
  color: #f0a500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  margin: 0 0 16px 0;
  border-bottom: 1px solid rgba(240, 165, 0, 0.2);
  padding-bottom: 8px;
}

h3 {
  font-size: 13px;
  font-weight: 700;
  color: #00e5ff;
  margin: 20px 0 8px 0;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

h3:first-of-type { margin-top: 0; }

p {
  font-size: 13px;
  line-height: 1.65;
  margin: 0 0 10px 0;
  color: #a1b0c8;
}

ul {
  margin: 0 0 10px 0;
  padding-left: 20px;
  color: #a1b0c8;
}

li {
  font-size: 13px;
  line-height: 1.65;
  margin-bottom: 6px;
}

strong { color: #e8edf5; }
em { color: #c8d0e0; font-style: normal; }

/* Inline badge label */
.badge-inline {
  display: inline-block;
  background: rgba(0, 229, 255, 0.1);
  color: #00e5ff;
  border: 1px solid rgba(0, 229, 255, 0.25);
  border-radius: 3px;
  font-size: 10px;
  font-weight: 700;
  padding: 1px 6px;
  letter-spacing: 0.08em;
  vertical-align: middle;
  margin-left: 4px;
}

/* Color helpers */
.col-green { color: #10b981; font-weight: 700; }
.col-red   { color: #ef4444; font-weight: 700; }
.col-cyan  { color: #00e5ff; font-weight: 700; }
.col-amber { color: #f0a500; font-weight: 700; }

/* Regex display block */
.regex-block {
  font-family: 'Space Mono', monospace;
  font-size: 12px;
  background: #0d1520;
  border: 1px solid #1e2d3d;
  border-radius: 4px;
  padding: 10px 14px;
  color: #6a9fd8;
  word-break: break-all;
  line-height: 1.6;
  margin: 6px 0 10px 0;
}

/* Project details section */
.project-details {
  background: rgba(0, 229, 255, 0.03);
  border-color: rgba(0, 229, 255, 0.15);
}

.footer-credit {
  margin-top: 24px;
  padding-top: 16px;
  border-top: 1px dashed rgba(0, 229, 255, 0.25);
  font-size: 10px;
  color: #4a6a8a;
  text-align: center;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  line-height: 1.8;
}

/* Scrollbar */
::-webkit-scrollbar { width: 6px; }
::-webkit-scrollbar-track { background: #0a0a0f; }
::-webkit-scrollbar-thumb { background: #1e2d3d; border-radius: 3px; }
::-webkit-scrollbar-thumb:hover { background: #4a6a8a; }
</style>

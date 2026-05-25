<script setup>
</script>

<template>
  <div class="manual-plain">
    <div class="manual-header">
      <h1 class="manual-title">AutomaTe User Guide</h1>
      <div class="manual-subtitle">Interactive Computation & Visualization</div>
    </div>

    <div class="manual-content">
      <section>
        <h2>INTRODUCTION</h2>
        <p>
          Welcome to <strong>AutomaTe</strong>. This web-based platform serves as an interactive environment designed for exploring computation theory. By selecting complex regular expressions, users can instantly generate their corresponding mathematical models, transforming abstract logic into intuitive, graphical workflows. Our engine fully supports Deterministic Finite Automata (DFA), Context-Free Grammars (CFG), and Pushdown Automata (PDA).
        </p>
      </section>

      <section>
        <h2>GETTING STARTED</h2>
        
        <h3>Compatibility</h3>
        <p>
          AutomaTe runs entirely within your browser and requires no installation. While it supports all modern browsers (Chrome, Edge, Firefox), we highly recommend using a desktop environment to ensure the complex diagrams scale correctly.
        </p>
        
        <h3>Basic Workflow</h3>
        <ul>
          <li><strong>Select a Problem:</strong> Navigate to the left sidebar to choose from a list of predefined Regular Expression challenges.</li>
          <li><strong>Input Testing:</strong> Use the Test Terminal at the bottom of the screen to type out your target string (e.g., babababbaba).</li>
          <li><strong>Run Simulation:</strong> Hit enter or press the simulate button. AutomaTe will process the sequence and immediately update the visualizations based on your input.</li>
        </ul>
      </section>

      <section>
        <h2>CORE VISUALIZATIONS</h2>
        
        <h3>State Map (DFA)</h3>
        <p>This module renders the selected regular expression into a node-based network.</p>
        <ul>
          <li><strong>Nodes:</strong> Represent system states. The starting point is marked with a <strong>"-"</strong>, while successful accepting states are marked with a <strong>"+"</strong>.</li>
          <li><strong>Pathing:</strong> Directional arrows dictate how specific characters move the system from one state to another.</li>
          <li><strong>Execution Trace:</strong> Processing a string evaluates the entire network, instantly highlighting the outcome to show if the input was accepted or trapped.</li>
        </ul>

        <h3>CFG Derivation</h3>
        <p>This section breaks down the active language into hierarchical rules.</p>
        <ul>
          <li><strong>Grammar Rules:</strong> View all non-terminals, terminals, and transformation laws for the current problem.</li>
          <li><strong>Live Tracing:</strong> When a valid string is input, a step-by-step trace breaks down the leftmost derivation, revealing exactly how the grammar constructs the input.</li>
        </ul>

        <h3>PDA Architecture</h3>
        <p>The Pushdown Automaton module illustrates the underlying stack memory logic.</p>
        <ul>
          <li><strong>Static Model:</strong> Analyzes the layout of the PDA associated with the regex, detailing the push, pop, and read mechanics through a static structural diagram.</li>
        </ul>
      </section>

      <section>
        <h2>TROUBLESHOOTING</h2>
        <ul>
          <li><strong>Performance:</strong> Very complex strings or deeply nested automata models may require a brief moment to compute.</li>
          <li><strong>Failed Derivations:</strong> If a string is flagged as "Rejected" or highlights in red within the CFG trace, it violates the rules of the selected Regular Expression. Refer to the DFA pathing or the exact failure point in the CFG trace to debug.</li>
        </ul>
      </section>

      <section class="project-details">
        <h2>PROJECT DETAILS</h2>
        <p><strong>Purpose:</strong> Created for academic purposes for the S-CSPC327 - Theory of Automata and Formal Languages course.</p>
        <p><strong>Development Team:</strong> Gabriel Castillo (Lead Developer), Marc Daniel Cajayon, Luis Miguel Trias, Gian Paolo Ybut (Developers).</p>
        <p><strong>Technology Stack:</strong> Framework: Vue.js; Visualization: D3.js; Version Control: Git / GitHub.</p>
        
        <div class="footer-credit">
          AUTOMATA THEORY AND FORMAL LANGUAGES DLSUD<br>
          developed by Gabriel Castillo, Marc Daniel Cajayon, Luis Miguel Trias, Gian Paolo Ybut
        </div>
      </section>
    </div>
  </div>
</template>

<style scoped>
.manual-plain {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  padding: 32px 40px;
  background: #0a0a0f;
  color: #c8d0e0;
  font-family: 'Space Mono', monospace;
  overflow-y: auto;
}

.manual-header {
  border-bottom: 1px solid #1e2d3d;
  padding-bottom: 24px;
  margin-bottom: 32px;
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
  font-size: 14px;
  color: #8b9bb4;
  letter-spacing: 0.05em;
}

.manual-content {
  max-width: 800px;
  display: flex;
  flex-direction: column;
  gap: 32px;
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
  font-size: 14px;
  font-weight: 700;
  color: #00e5ff;
  margin: 20px 0 10px 0;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

p {
  font-size: 13px;
  line-height: 1.6;
  margin: 0 0 12px 0;
  color: #a1b0c8;
}

ul {
  margin: 0;
  padding-left: 20px;
  color: #a1b0c8;
}

li {
  font-size: 13px;
  line-height: 1.6;
  margin-bottom: 8px;
}

strong {
  color: #e8edf5;
}

.project-details {
  background: rgba(0, 229, 255, 0.03);
  border-color: rgba(0, 229, 255, 0.2);
}

.footer-credit {
  margin-top: 32px;
  padding-top: 16px;
  border-top: 1px dashed rgba(0, 229, 255, 0.3);
  font-size: 11px;
  color: #8b9bb4;
  text-align: center;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  line-height: 1.8;
}

/* Custom Scrollbar */
::-webkit-scrollbar {
  width: 6px;
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

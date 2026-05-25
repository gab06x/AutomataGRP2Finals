<script setup>
import { ref, watch, onMounted, computed, onUnmounted } from 'vue'
import * as d3 from 'd3'

const props = defineProps({
    problemId: { type: Number, required: true },
    testString: { type: String, default: '' },
    simKey: { type: Number, default: 0 }
})

const svgRef = ref(null)
const stepIndex = ref(0)
const isRunning = ref(false)
const done = ref(false)
const simResult = ref(null)
const autoTimer = ref(null)

const DFA_CONFIGS = {
  1: {
    start: 'q0',
    accept: ['q_accept'],
    nodes: [
      { id: 'q0',       label: '-', type: 'start',  fx: 0,    fy: 0   },
      { id: 'q1',       label: '',  type: 'state',  fx: 130,  fy: 0   },
      { id: 'q2',       label: '',  type: 'state',  fx: 260,  fy: 0   },
      { id: 'q3',       label: '',  type: 'state',  fx: 390,  fy: 0   },
      { id: 'q4',       label: '',  type: 'state',  fx: 520,  fy: 0   },
      { id: 'q5',       label: '',  type: 'state',  fx: 650,  fy: 0   },
      { id: 'q6',       label: '',  type: 'state',  fx: 780,  fy: 0   },
      { id: 'q7',       label: '',  type: 'state',  fx: 910,  fy: 0   },
      { id: 'q8',       label: '',  type: 'state',  fx: 1040, fy: 0   },
      { id: 'q9',       label: '',  type: 'state',  fx: 1170, fy: -80 },
      { id: 'q10',      label: '',  type: 'state',  fx: 1170, fy: 80  },
      { id: 'q11',      label: '',  type: 'state',  fx: 1300, fy: -80 },
      { id: 'q12',      label: '',  type: 'state',  fx: 1300, fy: 80  },
      { id: 'q_accept', label: '+', type: 'accept', fx: 1430, fy: 0   },
      { id: 'q_trap',   label: '',  type: 'state',  fx: 130,  fy: 130 }
    ],
    links: [
      // From q0
      { source: 'q0',       target: 'q_trap',   label: 'a',   sweep: 1 },
      { source: 'q0',       target: 'q1',        label: 'b'             },
      // From q1 — both a and b go to q2
      { source: 'q1',       target: 'q2',        label: 'a, b'          },
      // From q2
      { source: 'q2',       target: 'q_trap',    label: 'a',   sweep: 1 },
      { source: 'q2',       target: 'q3',        label: 'b'             },
      // From q3
      { source: 'q3',       target: 'q4',        label: 'a'             },
      { source: 'q3',       target: 'q3',        label: 'b'             },
      // From q4
      { source: 'q4',       target: 'q4',        label: 'a'             },
      { source: 'q4',       target: 'q5',        label: 'b'             },
      // From q5
      { source: 'q5',       target: 'q6',        label: 'a'             },
      { source: 'q5',       target: 'q5',        label: 'b'             },
      // From q6 — a goes to trap, b goes to q7
      { source: 'q6',       target: 'q_trap',    label: 'a',   sweep: 1 },
      { source: 'q6',       target: 'q7',        label: 'b'             },
      // From q7
      { source: 'q7',       target: 'q7',        label: 'a'             },
      { source: 'q7',       target: 'q8',        label: 'b'             },
      // From q8
      { source: 'q8',       target: 'q9',        label: 'a'             },
      { source: 'q8',       target: 'q10',       label: 'b'             },
      // From q9
      { source: 'q9',       target: 'q9',        label: 'a'             },
      { source: 'q9',       target: 'q11',       label: 'b'             },
      // From q10
      { source: 'q10',      target: 'q12',       label: 'a'             },
      { source: 'q10',      target: 'q10',       label: 'b'             },
      // From q11 (cross to accept and q10)
      { source: 'q11',      target: 'q_accept',  label: 'a'             },
      { source: 'q11',      target: 'q10',       label: 'b',  sweep: 1  },
      // From q12 (cross to q9 and accept)
      { source: 'q12',      target: 'q9',        label: 'a',  sweep: 0  },
      { source: 'q12',      target: 'q_accept',  label: 'b'             },
      // Trap and accept self-loops
      { source: 'q_trap',   target: 'q_trap',    label: 'a, b'          },
      { source: 'q_accept', target: 'q_accept',  label: 'a, b'          }
    ],
    transitions: {
      'q0':       { 'a': 'q_trap',   'b': 'q1'       },
      'q1':       { 'a': 'q2',       'b': 'q2'       },
      'q2':       { 'a': 'q_trap',   'b': 'q3'       },
      'q3':       { 'a': 'q4',       'b': 'q3'       },
      'q4':       { 'a': 'q4',       'b': 'q5'       },
      'q5':       { 'a': 'q6',       'b': 'q5'       },
      'q6':       { 'a': 'q_trap',   'b': 'q7'       },
      'q7':       { 'a': 'q7',       'b': 'q8'       },
      'q8':       { 'a': 'q9',       'b': 'q10'      },
      'q9':       { 'a': 'q9',       'b': 'q11'      },
      'q10':      { 'a': 'q12',      'b': 'q10'      },
      'q11':      { 'a': 'q_accept', 'b': 'q10'      },
      'q12':      { 'a': 'q9',       'b': 'q_accept' },
      'q_trap':   { 'a': 'q_trap',   'b': 'q_trap'   },
      'q_accept': { 'a': 'q_accept', 'b': 'q_accept' }
    }
  },

  2: {
    start: 'q0',
    accept: ['q_accept2'],
    nodes: [
      // Start state — far left
      { id: 'q0',       label: '-', type: 'start',  fx: 0,   fy: 0    },
      // Left diamond — upper path via '1'
      { id: 'q1',       label: '',  type: 'state',  fx: 160, fy: -100 },
      { id: 'q2',       label: '',  type: 'state',  fx: 300, fy: -190 },
      { id: 'q3',       label: '',  type: 'state',  fx: 300, fy: -100 },
      // Left diamond — lower path via '0'
      { id: 'q4',       label: '',  type: 'state',  fx: 160, fy: 100  },
      // Bottleneck center
      { id: 'q5',       label: '',  type: 'state',  fx: 440, fy: 0    },
      // Right structure — upper branch
      { id: 'q6',       label: '',  type: 'state',  fx: 580, fy: -110 },
      // Right structure — lower branch (self-loop on 1)
      { id: 'q7',       label: '',  type: 'state',  fx: 580, fy: 110  },
      // Second-right structure
      { id: 'q8',       label: '',  type: 'state',  fx: 720, fy: -50  },
      { id: 'q9',       label: '',  type: 'state',  fx: 720, fy: 110  },
      // Accept state — far right
      { id: 'q_accept2', label: '+', type: 'accept', fx: 860, fy: 0   }
    ],
    links: [
      // From start
      { source: 'q0',        target: 'q1',        label: '1'           },
      { source: 'q0',        target: 'q4',        label: '0'           },
      // From q1
      { source: 'q1',        target: 'q2',        label: '0'           },
      { source: 'q1',        target: 'q3',        label: '1'           },
      // From q2 — both 0 and 1 go to q5
      { source: 'q2',        target: 'q5',        label: '0, 1'        },
      // From q3 — 0 goes back to q2, 1 goes to q5
      { source: 'q3',        target: 'q2',        label: '0'           },
      { source: 'q3',        target: 'q5',        label: '1'           },
      // From q4
      { source: 'q4',        target: 'q5',        label: '0'           },
      { source: 'q4',        target: 'q1',        label: '1', sweep: 0 },
      // From q5
      { source: 'q5',        target: 'q6',        label: '0'           },
      { source: 'q5',        target: 'q7',        label: '1'           },
      // From q6
      { source: 'q6',        target: 'q8',        label: '0'                        },
      { source: 'q6',        target: 'q_accept2', label: '1', sweep: 1, curve: -1 },
      // From q7 — self-loop on 1
      { source: 'q7',        target: 'q7',        label: '1'           },
      { source: 'q7',        target: 'q9',        label: '0'           },
      // From q8 — both go to accept
      { source: 'q8',        target: 'q_accept2', label: '0, 1'        },
      // From q9
      { source: 'q9',        target: 'q8',        label: '0'           },
      { source: 'q9',        target: 'q_accept2', label: '1'           },
      // Accept self-loop
      { source: 'q_accept2', target: 'q_accept2', label: '0, 1'        }
    ],
    transitions: {
      'q0':       { '0': 'q4',        '1': 'q1'        },
      'q1':       { '0': 'q2',        '1': 'q3'        },
      'q2':       { '0': 'q5',        '1': 'q5'        },
      'q3':       { '0': 'q2',        '1': 'q5'        },
      'q4':       { '0': 'q5',        '1': 'q1'        },
      'q5':       { '0': 'q6',        '1': 'q7'        },
      'q6':       { '0': 'q8',        '1': 'q_accept2' },
      'q7':       { '0': 'q9',        '1': 'q7'        },
      'q8':       { '0': 'q_accept2', '1': 'q_accept2' },
      'q9':       { '0': 'q8',        '1': 'q_accept2' },
      'q_accept2':{ '0': 'q_accept2', '1': 'q_accept2' }
    }
  }
}

const REGEX_MAP = {
  1: '(bab+bbb) b* a* (a*+b*) (ab)* (aba) (bab+aba)* bb (a+b)* (bab+aba) (a+b)*',
  2: '(1+0)* 0* 1* (111+00+101) (1+0)* (101+01+000) (1+0)* (101+000)*'
}

const dfa = computed(() => DFA_CONFIGS[props.problemId])
const problemRegex = computed(() => REGEX_MAP[props.problemId])

const steps = computed(() =>
  simResult.value ? simResult.value.steps : [{ state: dfa.value.start, charIndex: -1, char: null }]
)

const currentStep = computed(() => steps.value[stepIndex.value] || steps.value[steps.value.length - 1])
const currentState = computed(() => currentStep.value?.state ?? null)
const currentCharIdx = computed(() => simResult.value ? (steps.value[stepIndex.value]?.charIndex ?? -1) : -1)

const resultAccepted = computed(() => done.value && !!simResult.value?.accepted)

const isValidInput = computed(() => {
    if (props.testString === null || props.testString === undefined) return false;
    const result = runSimulation(props.testString)
    return result.accepted
})

const tape = computed(() => {
  if (!props.testString) return []
  return props.testString.split('').map((ch, i) => {
    const idx = currentCharIdx.value
    if (!simResult.value) return { ch, status: 'pending' }
    if (i < idx) return { ch, status: 'done' }
    if (i === idx) return { ch, status: 'active' }
    return { ch, status: 'pending' }
  })
})

const runSimulation = (input) => {
  const d = dfa.value
  const stepsList = [{ state: d.start, charIndex: -1, char: null }]
  let current = d.start
  for (let i = 0; i < input.length; i++) {
    const ch = input[i]
    let next = d.transitions[current]?.[ch]

    // Handle split transitions string like "0, 1"
    if (!next) {
        for (const [key, val] of Object.entries(d.transitions[current])) {
            if (key.includes(ch)) next = val;
        }
    }

    if (next === undefined) {
      stepsList.push({ state: null, charIndex: i, char: ch, dead: true })
      return { steps: stepsList, accepted: false }
    }
    current = next
    stepsList.push({ state: current, charIndex: i, char: ch })
  }
  return { steps: stepsList, accepted: d.accept.includes(current) }
}

const initSim = () => {
  const result = runSimulation(props.testString)
  simResult.value = result
  return result
}

const highlightElements = () => {
    const svg = d3.select(svgRef.value)
    // Reset all
    svg.selectAll('rect.node-rect')
      .attr('stroke', '#2d4a6b')
      .attr('stroke-width', 1.5)
      .style('filter', null)
    svg.selectAll('path.edge')
      .attr('stroke', '#2d4a6b')
      .attr('stroke-width', 1.5)

    if (!simResult.value || !steps.value) return;

    const isAccepted = resultAccepted.value;
    const isDone = done.value;
    const activeColor = isDone ? (isAccepted ? '#c8ff00' : '#ff4444') : '#00e5ff';
    const trailColor = isDone ? (isAccepted ? '#c8ff00' : '#ff4444') : '#32ff7e';

    // Highlight all visited paths (the trail)
    for (let i = 0; i <= stepIndex.value; i++) {
        const step = steps.value[i];
        if (!step) continue;
        
        const stateId = step.state;
        
        // Highlight edge from previous to current
        if (i > 0) {
            const prevStep = steps.value[i - 1];
            if (prevStep && prevStep.state && stateId) {
                const pState = prevStep.state;
                svg.selectAll(`path[id^="link-${pState}-${stateId}-"]`)
                   .attr('stroke', trailColor)
                   .attr('stroke-width', 2);
            }
        }

        // Highlight node
        if (stateId) {
            const isLast = (i === stepIndex.value);
            const color = isLast ? activeColor : trailColor;
            const width = isLast ? 2.5 : 2;
            const filter = isLast ? `drop-shadow(0 0 10px ${color})` : null;
            
            svg.select(`#node-${stateId}`)
              .attr('stroke', color)
              .attr('stroke-width', width)
              .style('filter', filter);
        }
    }
    
    // If the last step is a dead state (null), color the last known state red
    const currentStepData = steps.value[stepIndex.value];
    if (currentStepData && currentStepData.dead) {
        const prevStep = steps.value[stepIndex.value - 1];
        if (prevStep && prevStep.state) {
            svg.select(`#node-${prevStep.state}`)
              .attr('stroke', '#ff4444')
              .attr('stroke-width', 2.5)
              .style('filter', `drop-shadow(0 0 10px #ff4444)`);
        }
    }
}

const advance = (result, idx) => {
  const from = result.steps[idx].state
  const to = result.steps[idx + 1]?.state
  stepIndex.value = idx + 1

  highlightElements()

  if (idx + 1 >= result.steps.length - 1) done.value = true
}

const runAuto = () => {
  if (props.testString === null || props.testString === undefined) return;
  doReset();
  const result = initSim()
  isRunning.value = true
  let idx = 0
  stepIndex.value = 0
  done.value = false

  highlightElements()

  const max = result.steps.length - 1
  autoTimer.value = setInterval(() => {
    if (idx >= max) {
      clearInterval(autoTimer.value)
      autoTimer.value = null
      isRunning.value = false
      done.value = true
      highlightElements()
      return
    }
    advance(result, idx)
    idx++
  }, 800)
}

const doReset = () => {
  clearInterval(autoTimer.value)
  autoTimer.value = null
  isRunning.value = false
  stepIndex.value = 0
  done.value = false
  simResult.value = null

  if (svgRef.value) {
      d3.select(svgRef.value).selectAll('rect.node-rect')
        .attr('stroke', '#2d4a6b')
        .attr('stroke-width', 1.5)
        .style('filter', null)
      d3.select(svgRef.value).selectAll('path.edge')
        .attr('stroke', '#2d4a6b')
        .attr('stroke-width', 1.5)
  }
}

const NODE_W = 46
const NODE_H = 30
const NODE_R = 5

const renderDFA = () => {
    if (!svgRef.value) return;

    const data = dfa.value;
    d3.select(svgRef.value).selectAll("*").remove();

    const svg = d3.select(svgRef.value).style("overflow", "hidden");

    // Arrow marker - cyan
    const defs = svg.append("defs")

    defs.append("marker")
        .attr("id", "arrow-default")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX", 24)
        .attr("refY", 0)
        .attr("markerWidth", 5)
        .attr("markerHeight", 5)
        .attr("orient", "auto")
        .append("path")
        .attr("d", "M0,-5L10,0L0,5")
        .attr("fill", "#2d4a6b");

    defs.append("marker")
        .attr("id", "arrow-active")
        .attr("viewBox", "0 -5 10 10")
        .attr("refX", 24)
        .attr("refY", 0)
        .attr("markerWidth", 5)
        .attr("markerHeight", 5)
        .attr("orient", "auto")
        .append("path")
        .attr("d", "M0,-5L10,0L0,5")
        .attr("fill", "#00e5ff");

    const simulation = d3.forceSimulation(data.nodes)
        .force("link", d3.forceLink(data.links).id(d => d.id));

    // Links
    const linkGroup = svg.append("g")
    const link = linkGroup.selectAll("path")
        .data(data.links)
        .join("path")
        .attr("class", "edge")
        .attr("id", d => `link-${d.source.id ?? d.source}-${d.target.id ?? d.target}-${d.label}`)
        .attr("fill", "none")
        .attr("stroke", "#2d4a6b")
        .attr("stroke-width", 1.5)
        .attr("marker-end", "url(#arrow-default)");

    // Link labels
    const linkLabel = svg.append("g").selectAll("text")
        .data(data.links).join("text").text(d => d.label)
        .attr("font-size", "11px")
        .attr("fill", "#e06b75")
        .attr("font-weight", "700")
        .attr("font-family", "Space Mono, monospace")
        .attr("text-anchor", "middle")
        .style("paint-order", "stroke")
        .style("stroke", "#0d0f18")
        .style("stroke-width", "4px")
        .style("stroke-linejoin", "round");

    // Node groups
    const nodeGroup = svg.append("g")
    const nodeEl = nodeGroup.selectAll("g.node-g")
        .data(data.nodes)
        .join("g")
        .attr("class", "node-g")

    // Rounded rectangle for each node
    nodeEl.append("rect")
        .attr("class", "node-rect")
        .attr("id", d => `node-${d.id}`)
        .attr("width", NODE_W)
        .attr("height", NODE_H)
        .attr("rx", NODE_R)
        .attr("ry", NODE_R)
        .attr("x", -NODE_W / 2)
        .attr("y", -NODE_H / 2)
        .attr("fill", d => {
            if (d.type === 'accept') return '#0d1f0d'
            if (d.type === 'start') return '#1a1a0d'
            return '#111827'
        })
        .attr("stroke", d => {
            if (d.type === 'accept') return '#c8ff00'
            if (d.type === 'start') return '#4a6a8a'
            return '#2d4a6b'
        })
        .attr("stroke-width", 1.5)

    // Accept state double border
    nodeEl.filter(d => d.type === 'accept')
        .append("rect")
        .attr("width", NODE_W - 6)
        .attr("height", NODE_H - 6)
        .attr("rx", NODE_R - 1)
        .attr("ry", NODE_R - 1)
        .attr("x", -(NODE_W - 6) / 2)
        .attr("y", -(NODE_H - 6) / 2)
        .attr("fill", "none")
        .attr("stroke", "#c8ff00")
        .attr("stroke-width", 1)
        .attr("pointer-events", "none")

    // Node ID label — q_trap shows as "T", accept states show as "+", others show their id
    const nodeIdLabel = nodeEl.append("text")
        .text(d => {
            if (d.id === 'q_trap') return 'T'
            if (d.id === 'q_accept' || d.id === 'q_accept2') return '+'
            return d.id
        })
        .attr("dy", 4)
        .attr("text-anchor", "middle")
        .attr("font-size", d => d.id.length > 3 ? "8px" : "10px")
        .attr("font-family", "Space Mono, monospace")
        .attr("font-weight", "700")
        .attr("pointer-events", "none")
        .attr("fill", d => {
            if (d.type === 'accept') return '#c8ff00'
            if (d.type === 'start') return '#f0a500'
            if (d.id === 'q_trap') return '#a0a0c0'
            return '#c8d0e0'
        })

    // State type labels below nodes
    nodeEl.append("text")
        .text(d => d.type === 'start' ? 'START' : d.type === 'accept' ? 'ACCEPT' : '')
        .attr("dy", NODE_H / 2 + 12)
        .attr("text-anchor", "middle")
        .attr("font-size", "8px")
        .attr("font-family", "Space Mono, monospace")
        .attr("font-weight", "700")
        .attr("letter-spacing", "0.08em")
        .attr("pointer-events", "none")
        .attr("fill", d => d.type === 'accept' ? '#c8ff00' : '#4a6a8a')

    simulation.tick(300);

    link.attr("d", d => {
        const dx = d.target.x - d.source.x;
        const dy = d.target.y - d.source.y;

        if (d.source === d.target) {
            const size = d.curve ? 20 * d.curve : 20;
            const yOffset = d.curve ? 18 + (d.curve - 1)*20 : 18;
            return `M${d.source.x - 10},${d.source.y - yOffset} A ${size} ${size} 0 1 1 ${d.source.x + 10},${d.source.y - yOffset}`;
        }

        let dr = Math.sqrt(dx * dx + dy * dy);

        let finalSweep;
        if (d.sweep !== undefined) {
            finalSweep = d.sweep;
        } else {
            const sourceNum = parseInt((d.source.id ?? d.source).replace(/\D/g, '')) || 0;
            const targetNum = parseInt((d.target.id ?? d.target).replace(/\D/g, '')) || 0;
            finalSweep = sourceNum < targetNum ? 1 : 0;
        }

        if (d.curve && d.curve > 1000) {
            return `M${d.source.x},${d.source.y} L${d.target.x},${d.target.y}`;
        } else {
            dr = dr * (d.curve || 1.3);
            return `M${d.source.x},${d.source.y} A ${dr} ${dr} 0 0 ${finalSweep} ${d.target.x},${d.target.y}`;
        }
    });

    linkLabel
        .attr("x", d => {
            if (d.source === d.target) return d.source.x;
            return link.nodes()[data.links.indexOf(d)].getPointAtLength(0.5 * link.nodes()[data.links.indexOf(d)].getTotalLength()).x;
        })
        .attr("y", d => {
            if (d.source === d.target) return d.source.y - 45;
            return link.nodes()[data.links.indexOf(d)].getPointAtLength(0.5 * link.nodes()[data.links.indexOf(d)].getTotalLength()).y;
        });

    nodeEl.attr("transform", d => `translate(${d.x},${d.y})`);

    if (data.nodes.length > 0) {
        const minX = Math.min(...data.nodes.map(n => n.x));
        const maxX = Math.max(...data.nodes.map(n => n.x));
        const minY = Math.min(...data.nodes.map(n => n.y));
        const maxY = Math.max(...data.nodes.map(n => n.y));
        const padding = 100;
        const width = maxX - minX + padding * 2;
        const height = maxY - minY + padding * 2;
        svg.attr("viewBox", `${minX - padding} ${minY - padding} ${width} ${height}`)
           .style("width", "100%")
           .style("max-width", "100%")
           .style("height", "100%")
           .style("min-height", "380px");
    }
    simulation.stop();
};

watch(() => props.problemId, () => {
    doReset();
    renderDFA();
});

watch(() => props.testString, (newStr) => {
    if (newStr !== null && newStr !== undefined) {
        runAuto();
    } else {
        doReset();
    }
});

watch(() => props.simKey, () => {
    if (props.testString !== null && props.testString !== undefined) {
        runAuto();
    }
});

onMounted(() => {
    renderDFA();
});

onUnmounted(() => {
    clearInterval(autoTimer.value);
});
</script>

<template>
  <div class="dfa-wrap">

    <!-- Header bar -->
    <div class="dfa-header">
      <div class="header-left">
        <span class="badge">DFA</span>
        <span class="title">Problem {{ problemId }}</span>
      </div>
      <div class="header-right">
        <span class="leg-item"><span class="dot start-dot"></span>Start</span>
        <span class="leg-item"><span class="dot state-dot"></span>State</span>
        <span class="leg-item"><span class="dot accept-dot"></span>Accept</span>
      </div>
    </div>

    <!-- Regex row -->
    <div class="regex-wrap" v-if="problemRegex">
      <span class="regex-label">Regex</span>
      <code class="regex-code">{{ problemRegex }}</code>
    </div>

    <!-- Invalid warning -->
    <div v-if="simResult && !isValidInput" class="invalid-warning">
      <span>⚠ Invalid String for this Automaton</span>
    </div>

    <!-- Simulation status -->
    <div class="simulation-status-card" v-if="simResult">

      <!-- Tape -->
      <div v-if="tape.length > 0" class="tape-section">
        <div class="section-label">Input Tape</div>
        <div class="tape-container no-scrollbar-x">
          <div class="tape-cap">#</div>
          <div
            v-for="(cell, i) in tape"
            :key="i"
            :class="['tape-cell', cell.status]"
          >{{ cell.ch }}</div>
          <div class="tape-cap">#</div>
        </div>
      </div>

      <!-- State & Result -->
      <div class="status-row">
        <div class="current-state-box" v-if="currentState">
          <span class="slabel">Current State</span>
          <div :class="['state-badge', done ? (resultAccepted ? 'ok' : 'fail') : 'active']">
            {{ currentState }}
          </div>
        </div>

        <div class="read-char-box" v-if="currentStep?.char != null">
          <span class="slabel">Reading</span>
          <div class="char-badge">{{ currentStep.char }}</div>
        </div>

        <div class="result-banner-box">
          <transition name="pop">
            <div v-if="done" :class="['banner', resultAccepted ? 'banner-ok' : 'banner-fail']">
              <span v-if="resultAccepted">✓ String Accepted</span>
              <span v-else-if="props.testString === ''">✕ null string rejected</span>
              <span v-else>✕ String Rejected</span>
            </div>
          </transition>
        </div>
      </div>

    </div>

    <!-- SVG canvas -->
    <div class="viz-container">
      <svg ref="svgRef"></svg>
      <!-- Current state indicator overlay -->
      <div v-if="simResult && currentState" class="current-indicator">
        <span class="ci-dot"></span>
        <span class="ci-label">{{ currentState }}</span>
        <span class="ci-sub">CURRENT</span>
      </div>
    </div>

  </div>
</template>

<style scoped>
.dfa-wrap {
    display: flex;
    flex-direction: column;
    gap: 0;
    width: 100%;
    height: 100%;
    background: transparent;
    font-family: 'Space Mono', monospace;
}

/* Header */
.dfa-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 8px;
    padding: 12px 16px;
    background: rgba(13, 13, 20, 0.8);
    border-bottom: 1px solid #1a2535;
}

.header-left {
    display: flex;
    align-items: center;
    gap: 8px;
}

.badge {
    background: rgba(0, 229, 255, 0.1);
    color: #00e5ff;
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 0.1em;
    padding: 3px 8px;
    border-radius: 3px;
    border: 1px solid rgba(0, 229, 255, 0.25);
    text-transform: uppercase;
}

.title {
    font-size: 13px;
    font-weight: 700;
    color: #c8d0e0;
    letter-spacing: 0.05em;
}

.header-right {
    display: flex;
    align-items: center;
    gap: 12px;
}

.leg-item {
    display: flex;
    align-items: center;
    gap: 5px;
    font-size: 10px;
    color: #4a6a8a;
    letter-spacing: 0.06em;
    text-transform: uppercase;
}

.dot {
    width: 8px;
    height: 8px;
    border-radius: 2px;
    display: inline-block;
}

.start-dot  { background: #f0a500; }
.state-dot  { background: #2d6a9f; }
.accept-dot { background: #c8ff00; }

/* Regex */
.regex-wrap {
    display: flex;
    align-items: flex-start;
    gap: 8px;
    background: rgba(13, 15, 24, 0.9);
    border-bottom: 1px solid #1a2535;
    padding: 8px 16px;
}

.regex-label {
    font-size: 9px;
    font-weight: 700;
    color: #2d4a6b;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    padding-top: 2px;
    white-space: nowrap;
    flex-shrink: 0;
}

.regex-code {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: #6a9fd8;
    word-break: break-all;
    line-height: 1.6;
}

/* Simulation status */
.simulation-status-card {
    border-bottom: 1px solid #1a2535;
    background: rgba(10, 12, 20, 0.95);
    padding: 10px 16px;
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.section-label {
    font-size: 9px;
    font-weight: 700;
    color: #2d4a6b;
    text-transform: uppercase;
    letter-spacing: 0.12em;
    margin-bottom: 5px;
}

/* Tape */
.tape-container {
    display: flex;
    gap: 3px;
    overflow-x: auto;
    padding: 2px 0;
}

.tape-cap {
    min-width: 26px;
    height: 28px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'Space Mono', monospace;
    font-weight: 700;
    font-size: 12px;
    color: #2d4a6b;
    border: 1px solid #1a2535;
    border-radius: 3px;
    background: #0a0c14;
}

.tape-cell {
    min-width: 28px;
    height: 28px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 1px solid #1e2d3d;
    border-radius: 3px;
    font-family: 'Space Mono', monospace;
    font-weight: 700;
    font-size: 12px;
    transition: all 0.2s ease;
    background: #111827;
    color: #4a6a8a;
}

.tape-cell.done {
    background: rgba(0, 229, 255, 0.05);
    color: #00e5ff;
    border-color: rgba(0, 229, 255, 0.2);
}

.tape-cell.active {
    background: rgba(0, 229, 255, 0.15);
    color: #00e5ff;
    border-color: #00e5ff;
    transform: translateY(-2px);
    box-shadow: 0 0 10px rgba(0, 229, 255, 0.4);
}

/* Status row */
.status-row {
    display: flex;
    align-items: center;
    gap: 16px;
    flex-wrap: wrap;
}

.slabel {
    font-size: 9px;
    font-weight: 700;
    color: #2d4a6b;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    display: block;
    margin-bottom: 4px;
}

.state-badge {
    padding: 3px 10px;
    border-radius: 3px;
    font-weight: 700;
    font-size: 11px;
    letter-spacing: 0.08em;
    border: 1px solid transparent;
}

.state-badge.active {
    background: rgba(0, 229, 255, 0.1);
    color: #00e5ff;
    border-color: rgba(0, 229, 255, 0.3);
    box-shadow: 0 0 8px rgba(0, 229, 255, 0.2);
}

.state-badge.ok {
    background: rgba(200, 255, 0, 0.1);
    color: #c8ff00;
    border-color: rgba(200, 255, 0, 0.3);
    box-shadow: 0 0 8px rgba(200, 255, 0, 0.2);
}

.state-badge.fail {
    background: rgba(255, 68, 68, 0.1);
    color: #ff4444;
    border-color: rgba(255, 68, 68, 0.3);
}

.char-badge {
    padding: 3px 10px;
    background: rgba(240, 165, 0, 0.1);
    color: #f0a500;
    border: 1px solid rgba(240, 165, 0, 0.3);
    border-radius: 3px;
    font-weight: 700;
    font-family: 'Space Mono', monospace;
    font-size: 11px;
}

.banner {
    padding: 5px 12px;
    border-radius: 3px;
    font-size: 11px;
    font-weight: 700;
    letter-spacing: 0.06em;
}

.banner-ok   {
    background: rgba(200, 255, 0, 0.12);
    color: #c8ff00;
    border: 1px solid rgba(200, 255, 0, 0.3);
    box-shadow: 0 0 12px rgba(200, 255, 0, 0.2);
}

.banner-fail {
    background: rgba(255, 68, 68, 0.12);
    color: #ff4444;
    border: 1px solid rgba(255, 68, 68, 0.3);
}

/* Viz container */
.viz-container {
    flex: 1;
    position: relative;
    overflow: hidden;
    min-height: 320px;
}

.viz-container svg {
    width: 100%;
    height: 100%;
    min-height: 320px;
}

/* Current state indicator */
.current-indicator {
    position: absolute;
    bottom: 12px;
    left: 16px;
    display: flex;
    align-items: center;
    gap: 6px;
    background: rgba(0, 229, 255, 0.08);
    border: 1px solid rgba(0, 229, 255, 0.2);
    border-radius: 3px;
    padding: 4px 10px;
}

.ci-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #00e5ff;
    box-shadow: 0 0 6px #00e5ff;
    animation: blink 1s ease-in-out infinite;
}

@keyframes blink {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.3; }
}

.ci-label {
    font-size: 11px;
    font-weight: 700;
    color: #00e5ff;
    letter-spacing: 0.06em;
}

.ci-sub {
    font-size: 8px;
    color: #2d4a6b;
    letter-spacing: 0.1em;
    text-transform: uppercase;
    margin-left: 2px;
}

/* Invalid warning */
.invalid-warning {
    background: rgba(255, 68, 68, 0.08);
    color: #ff6b6b;
    padding: 6px 16px;
    border-bottom: 1px solid rgba(255, 68, 68, 0.2);
    font-size: 11px;
    font-weight: 500;
    letter-spacing: 0.04em;
}

/* Animations */
.pop-enter-active { animation: popIn 0.3s cubic-bezier(0.34, 1.56, 0.64, 1); }
@keyframes popIn {
  from { transform: scale(0.9); opacity: 0; }
  to   { transform: scale(1);    opacity: 1; }
}

.no-scrollbar-x { scrollbar-width: none; }
.no-scrollbar-x::-webkit-scrollbar { display: none; }
</style>

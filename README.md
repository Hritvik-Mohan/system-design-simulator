# System Design Simulator

> Draw a distributed system. Press Run. Watch it break — before you ship it.

A desktop application for simulating, stress-testing, and analysing high-level system designs using **Discrete Event Simulation (DES)**. Built on Electron + React, powered by a G/G/c/K queueing engine under the hood.

---

## What It Does

You drag nodes onto a canvas (API servers, databases, caches, load balancers), connect them with edges, configure traffic and failure scenarios, and then press **Run**. The engine simulates thousands of requests flowing through your architecture — tracking latency, queue depth, throughput, and cascading failures — all without a real server in sight.

The result: P50/P95/P99 latency breakdowns, per-node utilization heatmaps, request waterfall traces, failure cascade graphs, and cost estimates — before you write a line of production code.

```
┌──────────┐         ┌──────────┐         ┌──────────┐
│  Users   │────────►│ Gateway  │────────►│   API    │
│  source  │  https  │  lb-l7   │  grpc   │  micro   │
│ 980 rps  │  1ms    │ ██░░ 40% │  0.5ms  │ ████ 85% │
└──────────┘         └──────────┘         └─────┬────┘
                                                 │  tcp  2ms
                                          ┌──────▼────┐
                                          │    DB     │
                                          │ postgres  │
                                          │ ██████ 97%│  ← bottleneck
                                          └───────────┘
```

---

## Three Phases

### 1 — BUILD
Drag nodes from the palette onto the canvas. Connect them. Configure each node's queue parameters (workers, capacity, service time distribution, timeout), resilience settings (circuit breaker, rate limiter, retry policy), and SLO targets. Set up traffic patterns and fault injections in the scenario bar.

### 2 — SIMULATE
Press Run. The engine runs in a Web Worker — a discrete event loop that processes millions of events in time order, sampling service times from probability distributions (log-normal, exponential, Poisson, etc.). The canvas updates live: nodes shift from green to yellow to red as they saturate; edges pulse with traffic load.

### 3 — ANALYSE
When the simulation completes, a results tray expands with:
- **Summary** — P50 / P90 / P95 / P99 latency, throughput, error rate, availability, Little's Law check
- **Per-Node** — utilization, avg queue depth, RPS, rejection count, P99 per node
- **Traces** — waterfall views of individual requests (like Chrome DevTools' Network tab)
- **Failures** — causal cascade graph when failure injection triggers
- **Cost** — per-node and total cloud cost estimate (AWS / GCP / Azure)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Desktop shell | Electron 38 |
| Build system | electron-vite + Vite 7 |
| UI framework | React 19 + TypeScript 5 |
| Styling | Tailwind CSS 3 |
| Canvas | React Flow 11 |
| State management | Zustand 5 |
| Icons | Lucide React |
| Simulation engine | Discrete Event Simulation (DES) — planned Web Worker |

---

## Project Structure

```
system-design-simulator/
├── src/
│   ├── main/                          # Electron main process
│   │   ├── index.ts                   # App lifecycle, window creation
│   │   └── ipcHandlers.ts             # IPC bridge (file save/load)
│   ├── preload/                       # Context bridge
│   │   ├── index.ts
│   │   └── index.d.ts
│   └── renderer/src/                  # React app
│       ├── App.tsx
│       ├── main.tsx
│       ├── config/
│       │   ├── nodeRegistry.ts        # All node type definitions + defaults
│       │   ├── catalogConfig.ts       # Component catalogue metadata
│       │   ├── fieldConfig.tsx        # Inspector form field schemas
│       │   ├── node.ts                # Node shape types
│       │   └── themeConfig.ts         # Per-node-type colour themes
│       ├── store/
│       │   └── useStore.ts            # Zustand store (nodes, edges, file state)
│       ├── hooks/
│       │   ├── useFileHandlers.ts     # Open / save file via IPC
│       │   └── useFlowPersistence.ts  # Serialize canvas to JSON
│       ├── services/
│       │   └── FileService.ts         # File I/O abstraction
│       ├── types/
│       │   └── ui.ts                  # Shared UI types
│       ├── utils/
│       │   └── nodeTransformers.ts    # React Flow ↔ topology JSON conversions
│       └── components/
│           ├── atoms/                 # Primitive UI elements
│           │   ├── CtaButton.tsx
│           │   ├── Divider.tsx
│           │   ├── IconButton.tsx
│           │   ├── Input.tsx
│           │   ├── Label.tsx
│           │   ├── MenuHeader.tsx
│           │   ├── MenuOption.tsx
│           │   ├── MenuTrigger.tsx
│           │   ├── NodeHandle.tsx
│           │   ├── ProgressBar.tsx
│           │   ├── ResizeHandle.tsx
│           │   ├── Select.tsx
│           │   ├── Slider.tsx
│           │   ├── StatusBadge.tsx
│           │   ├── Switch.tsx
│           │   ├── ToggleButton.tsx
│           │   ├── TrafficParticle.tsx
│           │   └── UniversalHandle.tsx
│           ├── molecules/             # Composed UI blocks
│           │   ├── Branding.tsx
│           │   ├── FileStatus.tsx
│           │   ├── FormField.tsx
│           │   ├── LibraryItem.tsx
│           │   ├── MetricItem.tsx
│           │   ├── NodeHeader.tsx
│           │   ├── NodeSettingsMenu.tsx
│           │   ├── PropertiesNodeIcon.tsx
│           │   └── flow/edges/
│           │       └── PacketEdge.tsx # Animated traffic edge
│           ├── organisms/             # Full sections / panels
│           │   ├── FlowCanvas.tsx     # React Flow canvas (drag, drop, connect)
│           │   ├── Header.tsx
│           │   ├── LibrarySidebar.tsx # Node palette (drag source)
│           │   ├── PropertiesForm.tsx
│           │   ├── PropertiesHeader.tsx
│           │   └── PropertiesPanel.tsx
│           ├── features/
│           │   ├── ThemeToggle.tsx
│           │   ├── canvas/
│           │   │   ├── config/flowConfig.ts   # Edge types, grid config
│           │   │   ├── hooks/
│           │   │   │   ├── useFlowDnD.ts      # Drag-and-drop onto canvas
│           │   │   │   └── useFlowStore.ts    # Canvas ↔ Zustand bridge
│           │   │   └── utils/canvasUtils.ts
│           │   └── nodes/
│           │       ├── ComputeNode.tsx        # API server, lambda, worker, cron
│           │       ├── ServiceNode.tsx        # DB, cache, load balancer
│           │       └── vpc/
│           │           ├── VpcNode.tsx        # VPC / region boundary
│           │           ├── VpcHeader.tsx
│           │           ├── VpcToolBar.tsx
│           │           └── useVpcLogic.ts
│           └── templates/
│               └── WorkspaceLayout.tsx        # Three-panel layout
│
├── hld-simulator-docs/                # Git submodule — design & engine docs
│   ├── docs/                          # 5-part DES curriculum
│   ├── schema/                        # Full TypeScript type system (2300+ lines)
│   ├── canonical-catalogue/           # 17 CSV reference files
│   ├── planning/                      # 10-phase implementation plan + 46 tickets
│   └── design-decisions/              # Architecture Decision Records (ADRs)
│
├── build/                             # Electron Builder assets (icons, entitlements)
├── resources/                         # App icon
├── electron.vite.config.ts
├── electron-builder.yml
├── tailwind.config.js
├── tsconfig.json
└── package.json
```

---

## Node Types

| Node | Type | Description |
|---|---|---|
| API Server | `computeNode` | Long-running process, configurable CPU/queue |
| Serverless Fn | `computeNode` | Event-driven, low baseline utilization |
| Job Worker | `computeNode` | Background task processing |
| Cron Job | `computeNode` | Scheduled execution |
| Primary DB | `serviceNode` | Relational SQL datastore |
| Redis Cache | `serviceNode` | In-memory key/value store |
| Load Balancer | `serviceNode` | L7 request routing |
| VPC Region | `vpcNode` | Isolated network boundary / grouping |

---

## Simulation Engine (Planned)

The engine is a **Discrete Event Simulation loop** — no real clocks, no real servers, only a priority queue of timestamped events processed in order.

Each node is modelled as a **G/G/c/K queue**:
- `c` — concurrent workers
- `K` — max queue capacity (excess arrivals are rejected)
- Service time sampled from a configurable probability distribution (log-normal, exponential, Poisson, Weibull, etc.)

Key engine components being built (see `hld-simulator-docs/planning/`):

| Component | Role |
|---|---|
| Min-Heap | O(log n) event priority queue |
| SFC32 PRNG | Deterministic random (same seed = identical results every time) |
| G/G/c/K Node | Per-node queue model with workers and capacity |
| Workload Generator | Constant / Poisson / Spike / Diurnal / Bursty traffic |
| Network Edge | Latency distributions, congestion, packet loss |
| Failure Injector | Crash / latency spike / error rate faults at configurable times |
| Failure Propagation | Cascade walk through the dependency graph |
| Circuit Breaker | CLOSED / OPEN / HALF_OPEN state machine |
| Metrics Collector | Latency percentiles, throughput, error rate, Little's Law check |
| Request Tracer | Per-request waterfall data |
| Web Worker | Runs engine off the main thread; streams snapshots to UI |

---

## Submodule: `hld-simulator-docs`

The `hld-simulator-docs/` directory is a Git submodule containing all design documentation:

```
hld-simulator-docs/
├── docs/
│   ├── SYSTEM_OVERVIEW.md             # End-to-end system reference
│   ├── theoretical-foundations.md     # Queueing theory, DEVS, reliability
│   ├── 01-system-diagrams.md          # Nodes, edges, graph patterns
│   ├── 02-simulation-fundamentals.md  # Events, time, the event loop
│   ├── 03-data-structures-and-mechanics.md  # Min-heap, PRNG, G/G/c/K
│   ├── 04-distributed-systems-and-failures.md  # Network physics, failure modes
│   └── 05-devs-chaos-and-analysis.md  # DEVS formalism, chaos, output analysis
├── schema/
│   └── complete_simulator_schema.ts   # 2300+ line TypeScript type system
├── canonical-catalogue/               # 17 CSV reference files covering:
│   │                                  #   component taxonomy (110+ types)
│   │                                  #   failure modes & propagation rules
│   │                                  #   architectural patterns & anti-patterns
│   │                                  #   metrics & SLIs
│   │                                  #   pre-built scenarios
│   │                                  #   AWS / GCP / Azure provider mapping
│   └── README.md
├── planning/
│   ├── IMPLEMENTATION_PLAN.md         # 10-phase build plan
│   └── TICKETS.md                     # 46 engineering tickets with acceptance criteria
└── design-decisions/
    ├── adr-internal-modularity-over-plugin-system.md
    └── adr-no-custom-change-detection.md
```

To initialise the submodule after cloning:

```bash
git submodule update --init --recursive
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- npm

### Install

```bash
npm install
```

### Development

```bash
npm run dev
```

### Type check

```bash
npm run typecheck
```

### Build

```bash
# macOS
npm run build:mac

# Windows
npm run build:win

# Linux
npm run build:linux
```

---

## Design Principles

- **Deterministic by default** — every simulation run is seeded; the same seed always produces identical output
- **No decorative animation** — every visual (colour change, edge pulse, queue bar) represents real simulation data
- **Mathematical transparency** — metrics show their formula on hover (e.g. `utilization = activeWorkers / maxWorkers`)
- **Desktop-first** — minimum 1280px viewport; no mobile layout compromise
- **Single source of truth** — canvas, inspector panel, and JSON topology viewer all read from and write to one Zustand store

---

## Implementation Status

| Area | Status |
|---|---|
| React Flow canvas (nodes + edges) | Done |
| Drag-and-drop node palette | Done |
| Node types (Compute, Service, VPC) | Done |
| Atomic design system (atoms → organisms) | Done |
| Zustand topology store | Done |
| File save / load via Electron IPC | Done |
| Simulation engine (DES loop) | Planned |
| Inspector panel | Planned |
| Scenario bar (workload + faults + controls) | Planned |
| Web Worker + live canvas coloring | Planned |
| Results tray (summary, traces, failures, cost) | Planned |
| CLI (`dsds run / validate / compare`) | Planned |

See `hld-simulator-docs/planning/TICKETS.md` for the full 46-ticket breakdown.

# Meld — PRD v1.0

## One-liner

An infinite canvas OS where agents and humans meld cards into workflows.

## Background

Existing human-agent interfaces are fundamentally limited:

- **IM (Feishu/Telegram)**: Agents are second-class citizens via bot APIs. Linear message flow can't express parallel work.
- **Agent platforms (Manus)**: Great agent rendering, no social/collaboration ability.
- **Workflow editors (n8n/ComfyUI/Dify)**: Pre-defined DAGs. No improvisation, no cycles.
- **Agent managers (Paperclip)**: Task management view — humans manage, agents execute. Not collaboration.

Meld is different: **a spatial operating system where humans and agents are equal participants.**

## Core Design Principles

### 1. Stacklands Mental Model
- Agents are villagers, software/services are resources. Drag them together — things happen.
- Complexity emerges from simple rules. Cards move on their own — agents work in the background.

### 2. Unix Pipe Philosophy
- Every card is a stateless data-processing function: input → process → output.
- Connections are temporary, improvisational. No pre-designed workflows.
- Cycles allowed — agent cards judge termination by goal achievement.
- Composition is real-time — data flows the moment you connect.

### 3. Canvas > Agent
- The canvas is the agent's container, not the agent's client.
- The canvas engine manages all card lifecycles — it can create/destroy agents.
- All state, files, and agent processes live on the agent node.
- User devices are thin rendering terminals.

### 4. Card Duality
- Human-facing: visual rendering (web pages, charts, players, forms)
- Agent-facing: structured JSON (schema + intent metadata)
- Same card, two interfaces.

### 5. Agent as Canvas Participant
- Agents are aware they exist on an infinite canvas.
- Agents can CRUD cards and pipe connections via Canvas API.
- Agents can read the full canvas topology as context.
- Agents can self-organize: create sub-agent cards, restructure pipes, add resource cards as needed.

### 6. Human Control
- Pause/resume individual agent cards, pipe chains, or the entire canvas.
- On resume, agents receive pause duration and buffered data summary.
- Physical disconnection (pulling a pipe) = pause.
- Global pause turns canvas into pure editing mode.

## Architecture

```
┌──────────────────────────────────┐
│ User Device (Tauri App)          │
│ = Thin rendering terminal        │
│ - Canvas rendering               │
│ - User input                     │
│ - Local file upload              │
│ - Webview rendering              │
└──────────────┬───────────────────┘
               │ WSS
┌──────────────▼───────────────────┐
│ Agent Node = Canvas host         │
│                                  │
│ Canvas Engine (server-side)      │
│ ├── Canvas file storage          │
│ ├── Card state management        │
│ ├── Pipe data-flow scheduling    │
│ └── Agent process lifecycle      │
│                                  │
│ Everything is a card:            │
│ ├── file / directory card        │
│ ├── agent card (= OpenClaw topic)│
│ ├── webview card (URL+extractor) │
│ ├── note card (time-series log)  │
│ ├── markdown card (default)      │
│ └── ...                          │
│                                  │
│ Local resources:                 │
│ GitLab / filesystem / DB / Docker│
└──────────────────────────────────┘
```

## Card System

### Visual Spec
- Appearance: black rectangular border, white background
- Default collapsed size: 2 units wide × 3 units tall
- Canvas grid: sparse dot matrix, cards snap to grid points
- Default rendering: Markdown

### Card Metadata Schema (JSON)

```json
{
  "id": "uuid",
  "type": "markdown | file | directory | agent | webview | note",
  "intent": "string describing purpose",
  "input": {
    "schema": {},
    "accepts": ["text/markdown", "application/json"]
  },
  "output": {
    "schema": {},
    "produces": ["text/markdown", "application/json"]
  },
  "state": "running | paused | idle | error",
  "meta": {
    "title": "",
    "created": "ISO8601",
    "position": { "x": 0, "y": 0 },
    "size": { "w": 2, "h": 3 },
    "collapsed": true
  }
}
```

### Card Types

| Type | Description | Input | Output |
|------|-------------|-------|--------|
| markdown | Default card, renders Markdown | text | text |
| file | File on agent node | path | file content |
| directory | Directory on agent node | path | file listing |
| agent | OpenClaw topic session | upstream card outputs + meta | processed result |
| webview | Embedded web page + auto data extraction | URL | human: rendered page; agent: extracted JSON |
| note | Persistent time-series log | any card output | chronological records |

### Agent Card Special Behaviors
- Backed by an OpenClaw topic session
- Reads all upstream cards' output schemas and intents to auto-generate prompts
- Can create new agent cards (spawn new agent processes) from within the canvas
- In cycles, judges termination against its goal/objective field
- Displays token consumption and cost on the card

### Canvas API (available to agent cards as tools)

```
canvas.card.create({ type, intent, position?, connections? })
canvas.card.update(cardId, { meta, content })
canvas.card.delete(cardId)
canvas.card.list({ filter? })
canvas.card.read(cardId)

canvas.pipe.connect(sourceCardId, targetCardId)
canvas.pipe.disconnect(sourceCardId, targetCardId)
canvas.pipe.list({ cardId? })

canvas.layout.autoArrange({ scope? })
canvas.snapshot()
```

### Webview Card Data Extraction
- Layer 1 (universal): DOM → accessibility tree → structured text
- Layer 2 (semantic): agent extracts fields based on intent
- Layer 3 (adapters, optional): per-site schema mappings for high-frequency sites

## Pipe System

### Rules
- Any card output port → any card input port
- Real-time data flow, no "run" button
- Cycles allowed (termination judged by agent cards)
- One card can feed multiple pipes simultaneously (stateless, no conflict)

### Visualization
- Active pipe: border breathing animation
- Cycle iteration: working node color deepens + breathing effect
- Data direction: flow indicators on connection lines

### Note Card as External Node
- Any card can connect to a note card for persistent logging
- Note cards serve as global notebooks and cross-flow context bridges

## User Profile
- Chrome-profile-style local storage for user config, LLM API keys, webview login state
- Carried to agent node on connection

## Persistence
- One canvas = one local JSON file on agent node
- Contains: all card definitions, positions, connections, agent session refs
- A canvas file can be referenced as a card in another canvas (canvas nesting)

## Canvas Interaction
- Sparse dot matrix grid with snap alignment
- Pan / zoom navigation
- Drag to create connections
- Auto-arrange for card organization
- Double-click webview card → fullscreen → close returns to canvas

## Tech Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Client | Tauri 2.0 | Cross-platform (desktop+mobile), native webview, small bundle |
| Canvas rendering | Custom (Canvas 2D / WebGL) | Full control |
| Frontend framework | TBD | - |
| Server | Node.js | Consistent with OpenClaw ecosystem |
| Agent runtime | OpenClaw Gateway | Reuse existing agent capabilities |
| Protocol | WebSocket | Real-time bidirectional, extends OpenClaw protocol |
| Persistence | Local JSON files | Simple, version-controllable |

## MVP Scope: Multi-Agent Coding

### User Story
A developer opens the canvas, drags in a directory card pointing to a repo. Creates an agent card, connects it. Writes a markdown card describing the feature. Connects it to the agent. The agent outputs code changes into file cards. Another agent card is created for code review, connected to the output. Review fails → pipe loops back to the coding agent (cycle). Review passes → results flow to a note card as record. The agent may also create additional cards on its own as needed during the process.

### MVP Card Types
1. markdown card (requirements, notes)
2. file / directory card (code)
3. agent card (coding agent, review agent)
4. note card (process log)

### MVP NOT included
- webview card
- Multi-user collaboration
- Card marketplace / third-party cards
- Mobile (desktop first)
- Canvas nesting

## Naming

**Meld** — to fuse, to blend. Drag cards together, meld them, things happen.

---

*v1.0 — 2026-04-04*

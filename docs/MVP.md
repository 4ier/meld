# Meld MVP — Minimal Viable Canvas

## Goal

Validate one thing: **can a canvas of cards connected by pipes, driven by agents, produce working code?**

Everything else is deferred.

## Scope

A single HTML page (no Tauri, no server yet). One canvas. Four card types. Manual connections. One agent node (local OpenClaw).

## What the MVP looks like

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  markdown   │───→│   agent     │───→│    file      │
│  "add auth" │    │  (coder)    │    │  output.py   │
└─────────────┘    └──────┬──────┘    └──────────────┘
                          │
                          ↓
                   ┌─────────────┐
                   │    note     │
                   │  (log)      │
                   └─────────────┘
```

User writes a requirement in the markdown card. Connects it to an agent card. Agent reads the requirement, produces code, outputs to a file card. Note card logs everything.

## Tech (keep it stupid simple)

- **One HTML file** + vanilla JS (or single Vite project, zero framework)
- **Canvas 2D** for rendering the grid, cards, and pipes
- **WebSocket** to local OpenClaw gateway for agent card execution
- **No server** — canvas state is a JSON blob in localStorage (export/import as file)
- **No Tauri** — runs in any browser during MVP

## Cards to implement

### 1. Markdown Card
- Editable text area (contenteditable or textarea)
- Output: the text content as string
- Render: basic markdown → HTML

### 2. File Card
- Shows file content (read-only in MVP)
- Input: text content → writes to display
- Output: the displayed content
- Path is metadata only (agent node file I/O deferred to post-MVP)

### 3. Agent Card
- Connects to OpenClaw gateway via WebSocket
- On trigger: collects all upstream cards' outputs, sends as message to a topic
- Streams response back, writes to output
- Displays: model name, token count, cost, running/paused/idle state
- Goal field: editable text, used for cycle termination judgment

### 4. Note Card
- Append-only time-series log
- Every input appended with timestamp
- Scrollable, read-only

## Canvas interactions (MVP subset)

- [x] Dot grid background
- [x] Pan (drag empty space) and zoom (scroll wheel)
- [x] Create card (right-click context menu → type selector)
- [x] Move card (drag)
- [x] Resize card (drag corner)
- [x] Snap to grid
- [x] Connect cards (drag from output port to input port)
- [x] Disconnect (click pipe → delete)
- [x] Select card → edit content
- [x] Pause/resume agent card (toggle button)
- [ ] Auto-arrange (deferred)
- [ ] Canvas API for agents (deferred — agents only process data in MVP, don't restructure canvas)

## Data flow

1. User edits markdown card content
2. User manually triggers "send" (click play button on agent card, or connect triggers auto-send)
3. Agent card collects upstream outputs → sends to OpenClaw topic
4. Response streams back → appears in agent card output area
5. Output propagates to downstream cards (file card displays it, note card appends it)
6. If cycle exists: agent card checks goal → re-triggers upstream if not met

## Canvas file format (localStorage JSON)

```json
{
  "version": 1,
  "cards": [
    {
      "id": "uuid",
      "type": "markdown",
      "position": { "x": 2, "y": 1 },
      "size": { "w": 2, "h": 3 },
      "content": "Add authentication to the API",
      "meta": { "intent": "", "title": "Requirement" }
    }
  ],
  "pipes": [
    { "from": "card-uuid-1", "to": "card-uuid-2" }
  ]
}
```

## Success criteria

1. I can write a requirement in a markdown card
2. Connect it to an agent card that talks to my OpenClaw
3. Agent produces code that appears in a file card
4. Note card has a log of the interaction
5. I can pause and resume the agent
6. The whole thing fits in one HTML file I can open in a browser

## What I explicitly do NOT care about in MVP

- Pretty UI (functional > beautiful)
- Mobile
- Multiple agent nodes
- File system access (file card is just a display, not real I/O)
- Webview cards
- Agent self-modifying the canvas
- Authentication
- Collaboration
- Performance with 100+ cards

## Timeline

Build it. Ship it. Use it. Then decide what's next.

---

*v1.0 — 2026-04-04*

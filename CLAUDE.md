# CLAUDE.md

## Project Overview

**Golf Training Tracker Pro** is a single-page web application for tracking golf approach shot practice sessions. It allows golfers to record approach shots (green hit, miss, hole-in), transition to a putting phase, and review session results with historical statistics and charts. The UI is in **Brazilian Portuguese (pt-BR)**.

## Repository Structure

```
golf-approach-tracker/
├── index.html          # Main application (HTML + CSS + JS, ~1260 lines)
├── index (5).html      # Older version / backup (uploaded via GitHub)
├── index (6).html      # Older version / backup (uploaded via GitHub)
└── CLAUDE.md           # This file
```

This is a **zero-dependency, no-build** project. Everything lives in a single `index.html` file containing inline CSS and JavaScript. The only external dependency is **Chart.js v2** loaded via CDN (`<script>` tag).

## Architecture

### Single-File Application (`index.html`)

The app is structured in three inline sections within one HTML file:

1. **`<style>`** — All CSS (~690 lines). Uses CSS gradients, flexbox/grid layouts, and responsive design. No preprocessor or framework.
2. **HTML body** — Semantic sections for config, tracking, results, and history modal.
3. **`<script>`** — All JavaScript (~560 lines). Plain vanilla JS, no framework.

### Application Flow

1. **Configuration Phase** (`#configSection`) — User sets date, rounds, balls per round, distance (yards), and club type.
2. **Approach Tracking Phase** (`#trackingSection`) — User records each shot as:
   - `green` — Ball landed on the green
   - `miss` — Ball missed the green
   - `hole` — Ball went in the hole
3. **Putting Phase** (`#puttingSection`) — For each green hit, user records putt made/missed.
4. **Results Phase** (`#resultsSection`) — Per-round summary with green %, putting %.
5. **History Modal** (`#historyModal`) — Aggregated stats, line chart (green % over time), bar chart (avg green % per club).

### Key State Management

- Global `session` object holds all runtime state (current round, shots, counts, phase).
- `phase` property toggles between `'approach'` and `'putting'`.

### Data Persistence

- **localStorage** key: `golfSessions` — JSON array of all saved session objects.
- Each session stores: date, timestamp, distance, club, round counts, and detailed round data.

### External Dependencies

- **Chart.js v2** via CDN (`https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.9.4/Chart.min.js`) — Used for line and bar charts in the history modal.

## Key Functions Reference

| Function | Purpose |
|---|---|
| `startSession()` | Validates config inputs and initializes a new training session |
| `startNewRound()` | Resets UI and state for a new round within a session |
| `recordShot(type)` | Records an approach shot (`green`, `miss`, or `hole`) |
| `startPuttingPhase()` | Transitions from approach to putting; skips if no greens hit |
| `recordPutt(made)` | Records a putt result (boolean) |
| `finishRound()` | Saves round data; advances to next round or shows results |
| `undoLastAction()` | Undoes the last shot/putt recording |
| `showResults()` | Renders per-round summary and triggers `saveSession()` |
| `saveSession()` | Persists session data to localStorage |
| `showHistory()` | Opens history modal with stats, charts, and session list |
| `clearHistory()` | Clears all saved sessions from localStorage (with confirmation) |

## Development Workflow

### Running Locally

No build step required. Open `index.html` directly in a browser:

```bash
# Using Python's built-in server:
python3 -m http.server 8000

# Or simply open the file:
open index.html        # macOS
xdg-open index.html    # Linux
```

### Testing

There are no automated tests. All testing is manual via browser interaction.

### Making Changes

- All code changes go in `index.html`. There is no module system or build pipeline.
- CSS is at the top in a `<style>` block; JS is at the bottom in a `<script>` block.
- The files `index (5).html` and `index (6).html` are older uploaded versions and should generally not be modified.

## Conventions

- **Language**: UI text is in Brazilian Portuguese. Keep all user-facing strings in pt-BR.
- **Styling**: Inline CSS using CSS custom gradients (`linear-gradient(135deg, #667eea, #764ba2)` as the primary theme). No CSS framework.
- **JavaScript**: Vanilla JS only. No transpilation. DOM manipulation via `document.getElementById()`. No modules or imports.
- **Data format**: All measurements in yards. Club types include wedge variants (PW, GW, SW, LW) and irons (9i through 5i).
- **Chart.js**: Uses v2 API (legacy `xAxes`/`yAxes` syntax, not v3+). Keep chart code compatible with v2.

## Common Pitfalls

- Chart instances (`lineChart`, `barChart`) must be destroyed before re-creating to avoid canvas reuse errors.
- The `session` object is global and mutable — be careful with state when adding features.
- `localStorage` has no migration system; schema changes to `golfSessions` may break existing user data.

# PATHMIND v3.0

### AI Model Path Planning Arena — Terrain-Aware Multi-Algorithm Racing

> An interactive web app where 5 AI models (Claude, GPT-4o, Gemini Pro, ChatGPT, DeepSeek) race to navigate terrain-heavy obstacle courses using different pathfinding algorithms. Demonstrates robotics domain expertise, algorithm knowledge, and creative AI visualization.

---

## Live Demo

[**Open PathMind →**](https://nishanth3424.github.io/PathMind/)

---

## What is PathMind?

PathMind is a **terrain-aware pathfinding arena** where you design a 2D world with diverse terrain types — walls, uphill slopes, forests, mud, water, sand, and ice — then watch 5 AI models simultaneously race to find the best path. Each AI uses a fundamentally different algorithm, producing visually distinct paths that reveal the tradeoffs between speed, optimality, and exploration strategies.

---

## Features

### 8 Terrain Types with Distinct Costs
| Terrain | Cost | Visual | Description |
|---------|------|--------|-------------|
| **Empty** | 1.0× | Dark grid | Base movement cost |
| **Wall** | ∞ | Red glow | Impassable obstacle |
| **Uphill** | 3.0× | Green gradient ▲ | Steep incline, very slow |
| **Downhill** | 0.5× | Blue gradient ▼ | Descent advantage |
| **Forest** | 4.0× | Dark green trees | Dense cover, penalizes |
| **Mud** | 5.0× | Brown speckled | Swamp, highest passable cost |
| **Water** | ∞ | Animated blue waves | Impassable water body |
| **Sand** | 2.0× | Golden granular | Desert, moderate penalty |
| **Ice** | 0.8× | Light blue shimmer | Slightly faster than normal |

### 5 AI Model Racers
| AI Model | Algorithm | Behavior |
|----------|-----------|----------|
| **Claude** (Anthropic) | Weighted A* (Octile) | Methodical, terrain-aware, guaranteed optimal |
| **GPT-4o** (OpenAI) | A* (Euclidean, w=1.2) | Fast, slightly overestimates, trades optimality for speed |
| **Gemini Pro** (Google) | Goal-Biased RRT | Random sampling, creative paths through complex terrain |
| **ChatGPT** (OpenAI) | Dijkstra's Algorithm | No heuristic, explores everything, always finds cheapest path |
| **DeepSeek** | Greedy Best-First | Chases the goal aggressively, ignores terrain cost, can get trapped |

### 12 Preset Scenarios
- **Mountain Pass** — Uphill slopes with narrow passages
- **Swamp Maze** — Mud + water labyrinth
- **Frozen Lake** — Ice corridors with water hazards
- **Desert Crossing** — Sand dunes with uphill barriers
- **Volcanic Islands** — Lava rivers creating disconnected islands
- **Obstacle Course** — Sequential terrain challenges (gauntlet)
- **Forest Fortress** — Dense tree cover with walled center
- **Classic Maze** — Wall corridors (tight)
- **Scattered** — 15% random walls
- **Wall + Gap** — Single wall with narrow passage
- **U-Trap** — U-shaped trap around goal
- **Mixed Extreme** — All terrain types in chaos

### Interactive Features
- **Paint terrain** directly on the grid with 8 terrain brushes
- **AI Leaderboard** ranking all models by weighted path cost
- **Difficulty Meter** analyzing terrain complexity in real-time
- **Race Analysis** explaining why each AI chose its route
- **Configurable robot** size, animation speed, sensor range
- **Keyboard shortcuts** for rapid interaction

---

## How the Algorithms Work

### A* (Used by Claude, GPT-4o)
Best-first graph search guided by `f(n) = g(n) + w × h(n)`. Claude uses w=1.0 with octile heuristic (guaranteed optimal). GPT-4o uses w=1.2 with euclidean heuristic (faster but may miss optimal). Both account for terrain costs in g(n).

### Dijkstra's (Used by ChatGPT)
A* with h(n) = 0. Expands nodes purely by cost-so-far, exploring in all directions like a flood fill. Guaranteed optimal but explores far more nodes than A*.

### Greedy Best-First (Used by DeepSeek)
Uses only h(n), completely ignoring path cost. Blazingly fast in open terrain but can be fooled by terrain costs and dead ends.

### RRT (Used by Gemini Pro)
Rapidly-exploring Random Tree with 14% goal bias. Randomly samples positions and grows a tree toward them. Terrain-aware with greedy path smoothing. Produces creative, non-deterministic paths.

---

## Tech Stack

**Zero dependencies.** Pure HTML5 Canvas, vanilla JavaScript, hand-tuned CSS. All 5 algorithms, terrain cost engine, rendering, and analysis implemented from scratch. No build step, no frameworks, no CDN. Open the file → it runs.

---

## How to Run

1. Open `index.html` in any modern browser
2. Or deploy to GitHub Pages — it's a single HTML file
3. Or `npx serve .` for a local dev server

---

## Built By

**Nishanth** — B.S. Cyber-Physical Systems Engineering, University of Maryland, College Park '26

---

## License

[MIT](LICENSE)

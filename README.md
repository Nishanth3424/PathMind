# PATHMIND v4.0

### Terrain-Aware Pathfinding Arena — 5 Algorithms Race, Then a Robot Drives the Winner

> An interactive web app where 5 classic pathfinding algorithms — each themed as a well-known AI model — race across terrain-heavy maps. After the race, a physical robot **drives the winning route**, visibly slowing in mud, climbing uphill, and gliding on ice. Demonstrates CS fundamentals, robotics domain knowledge, and creative visualization.

---

## Live Demo

[**Open PathMind →**](https://nishanth3424.github.io/PathMind/)

---

## Honest note on the "AI models" (read this)

The five racers are named **Claude, GPT-4o, Gemini Pro, ChatGPT, and DeepSeek**, but to be completely clear:

- There are **no external API calls and no API keys.** Nothing is sent to any AI company.
- Each "model" is a **different classic pathfinding algorithm** running 100% offline in your browser.
- The AI-company names are a **memorable theme** for the algorithm tradeoffs — not a claim of AI integration.

This keeps the project honest and interview-safe: it's a strong *algorithms + visualization* demo, and you can describe it exactly that way.

| Racer (persona) | Real algorithm | Behavior |
|---|---|---|
| Claude | Weighted A* (octile heuristic) | Methodical, terrain-aware, guaranteed lowest-cost path |
| GPT-4o | A* (Euclidean, w=1.2) | Fast, slightly overestimates, trades a little optimality for speed |
| Gemini Pro | Goal-biased RRT | Random sampling, creative/unexpected routes |
| ChatGPT | Dijkstra's algorithm | No heuristic, explores everything, always optimal but slowest |
| DeepSeek | Greedy Best-First | Ignores terrain cost, charges at the goal, fast but often costlier |

---

## What's New in v4

### Terrain-Reactive Robot (the headline feature)
After a race, a physical robot **drives the winning path** and the terrain becomes real motion:
- **Crawls through mud** (0.3× speed) and sinks slightly, kicking up brown particles
- **Climbs uphill slowly** (0.4× speed) and visibly rises in elevation with a ground shadow
- **Rolls downhill fast** (1.6× speed) and drops in elevation
- **Glides on ice** (1.4× speed) and **slows on sand** (0.6×) and **forest** (0.45×)
- Leaves a glowing trail, faces its direction of travel, and reports live terrain + speed in a HUD
- Press **Drive Winner** (or `D`) anytime to replay it

### 8 Terrain Types with Distinct Costs & Drive Profiles
| Terrain | Path cost | Drive speed | Effect |
|---------|-----------|-------------|--------|
| Empty | 1.0× | 1.0× | Normal ground |
| Wall | ∞ (impassable) | — | Routed around |
| Uphill | 3.0× | 0.4× | Slow climb, robot rises |
| Downhill | 0.5× | 1.6× | Fast descent, robot drops |
| Forest | 4.0× | 0.45× | Slow push through trees |
| Mud | 5.0× | 0.3× | Crawl, robot sinks |
| Water | ∞ (impassable) | — | Routed around (robot can't swim) |
| Sand | 2.0× | 0.6× | Wheels slip |
| Ice | 0.8× | 1.4× | Low-friction glide |

> **On "traveling through water":** water and walls are **impassable** (infinite cost), so every algorithm routes *around* them and the driving robot only ever steps on valid cells. Paths are drawn between adjacent cells, so they never actually cross an obstacle.

### Plus everything from v3
- 12 presets (Mountain Pass, Swamp Maze, Frozen Lake, Desert Crossing, Volcanic Islands, Obstacle Course, Forest Fortress, Classic Maze, Scattered, Wall+Gap, U-Trap, Mixed Extreme)
- AI Leaderboard ranking models by weighted path cost
- Real-time Difficulty Meter
- Race Analysis explaining why each AI won/lost
- 8-brush terrain palette for painting maps

---

## How the Algorithms Work

- **A\*** — best-first search using `f = g + w·h`. Claude (w=1.0, octile) is optimal; GPT-4o (w=1.2, euclidean) is faster but can miss the true optimum.
- **Dijkstra's** — A* with `h = 0`; explores by cost only, always optimal, most nodes expanded.
- **Greedy Best-First** — uses only `h`, ignores path cost; fast but easily fooled by terrain/dead-ends.
- **RRT** — Rapidly-exploring Random Tree with 14% goal bias and greedy path smoothing; non-deterministic, creative routes.

All cost-aware searches factor terrain multipliers into `g(n)`.

---

## Tech Stack

**Zero dependencies.** Pure HTML5 Canvas + vanilla JavaScript + hand-tuned CSS. All algorithms, the terrain cost engine, the terrain-reactive robot physics, rendering, and analysis are implemented from scratch. No build step, no frameworks, no CDN, no API keys.

---

## Controls

`Space` — Race · `D` — Drive winner · `C` — Clear · `1-4` — Tools · `I` — About · `Esc` — Close

---

## How to Run

1. Open `index.html` in any modern browser, or
2. Deploy to GitHub Pages (single HTML file), or
3. `python -m http.server` / `npx serve .` for a local server

---

## Built By

**Nishanth** — B.S. Cyber-Physical Systems Engineering, University of Maryland, College Park '26

## License

[MIT](LICENSE)

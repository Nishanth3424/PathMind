# PathMind — Full Build Prompt for Claude Opus 4.6

> **Copy this entire prompt into a new Claude conversation using Opus 4.6.**
> **Estimated build time: 1–2 sessions. Cost: $0 (all tools/libraries are free and CDN-loaded).**

---

## PROMPT START — COPY EVERYTHING BELOW THIS LINE

---

You are building **PathMind** — a portfolio-grade, interactive web application for my LinkedIn and GitHub. I am a freshly graduated Cyber-Physical Systems Engineer from the University of Maryland (College Park), specializing in embedded systems and robotics. This project must demonstrate: robotics/CPS domain expertise, advanced AI integration, creative UI/UX thinking, and computer vision via MediaPipe hand tracking.

This is a **single-file React (.jsx) artifact** that runs entirely client-side. No backend. No paid services. Everything uses free CDN libraries. The Claude API calls will use the built-in Anthropic API access available inside artifacts.

---

## 1. PROJECT OVERVIEW

**PathMind** is an AI-powered robot path planning visualizer with hand-gesture controls. The user interacts with a 2D grid world where they can:

- Draw/erase obstacle walls with mouse OR with their hand via webcam (MediaPipe)
- Place a robot (start) and a goal (end)
- Configure robot parameters (size, speed, sensor range)
- Trigger pathfinding — the app runs A* and RRT algorithms simultaneously, visualizes both paths, and then sends the grid state + results to Claude API (Sonnet) which returns a plain-English robotics tutor explanation of WHY each algorithm chose its path, tradeoffs, and which is better for this specific scenario
- Drag obstacles in real-time and watch the path re-plan live with AI re-analysis

---

## 2. TECH STACK (ALL FREE, ALL CDN)

- **React** (built-in artifact support)
- **Tailwind CSS** (built-in utility classes)
- **Recharts** — for any data visualizations (path length comparison bar chart, algorithm performance metrics)
- **MediaPipe Hands** — `@mediapipe/hands` + `@mediapipe/camera_utils` via CDN for real-time hand tracking through webcam
- **Claude API** — use the built-in `fetch("https://api.anthropic.com/v1/messages", ...)` with model `claude-sonnet-4-20250514` for the AI tutor explanations (no API key needed inside artifacts)
- **lucide-react** — for icons (Bot, Target, Hand, Play, Trash, Settings, Github, etc.)

---

## 3. DETAILED UI LAYOUT & DESIGN DIRECTION

### Aesthetic: "Control Room Noir"
A dark, sophisticated, industrial-meets-futuristic aesthetic. Think NASA mission control crossed with a robotics lab terminal. NOT generic dark mode — this should feel like classified software.

- **Background**: Deep charcoal (#0a0a0f) with subtle scan-line overlay effect (CSS repeating-linear-gradient, 1px lines at 10% opacity)
- **Primary accent**: Electric cyan (#00e5ff) for paths, active elements, the robot
- **Secondary accent**: Warm amber (#ffab00) for the goal, warnings, RRT path
- **Danger/obstacles**: Muted red (#ff1744 at 60% opacity)
- **Font**: `"JetBrains Mono"` from Google Fonts for everything (monospace = control room feel). Import via `@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;700&display=swap')`
- **Grid cells**: Dark tiles with 1px border (#1a1a2e), obstacles are solid with a subtle inner glow
- **Cards/panels**: Background #111122 with 1px border of accent color at 20% opacity, no rounded corners (sharp = industrial), subtle box-shadow with accent color glow on hover
- **The robot**: A small animated SVG — a circle with a directional triangle inside, pulsing cyan glow (CSS animation)
- **The goal**: Amber diamond shape, gentle rotation animation
- **Paths**: A* path = solid cyan line with glow filter, RRT path = dashed amber line

### Layout (Desktop-first, responsive is nice but not critical)

```
┌─────────────────────────────────────────────────────────────┐
│  HEADER: "PATHMIND" logo (left) | Mode Toggle (right)       │
│          Hand 🤚 / Mouse 🖱️  | GitHub link                  │
├──────────────────────┬──────────────────────────────────────┤
│                      │  RIGHT PANEL (scrollable)             │
│                      │  ┌──────────────────────────────────┐ │
│                      │  │ CONTROL PANEL                    │ │
│                      │  │ • Robot Size slider (1-5 cells)  │ │
│                      │  │ • Speed slider (visual only)     │ │
│                      │  │ • Sensor Range slider (1-15)     │ │
│                      │  │ • Algorithm selector toggles     │ │
│   GRID CANVAS        │  │   [A*] [RRT] [Both]             │ │
│   (main area)        │  │ • [▶ PLAN PATH] button           │ │
│   30x30 grid         │  │ • [🗑 CLEAR ALL] button          │ │
│   with robot, goal,  │  └──────────────────────────────────┘ │
│   obstacles, paths   │  ┌──────────────────────────────────┐ │
│                      │  │ METRICS PANEL                    │ │
│                      │  │ • A* path length / nodes explored│ │
│                      │  │ • RRT path length / iterations   │ │
│                      │  │ • Recharts bar chart comparing   │ │
│                      │  └──────────────────────────────────┘ │
│                      │  ┌──────────────────────────────────┐ │
│                      │  │ AI TUTOR PANEL                   │ │
│                      │  │ Claude's explanation of the      │ │
│                      │  │ paths, tradeoffs, which is       │ │
│                      │  │ better and why. Typewriter        │ │
│                      │  │ effect for the text.             │ │
│                      │  └──────────────────────────────────┘ │
│                      │  ┌──────────────────────────────────┐ │
│                      │  │ HAND TRACKING STATUS             │ │
│                      │  │ • Webcam preview (small)         │ │
│                      │  │ • Detected gesture label         │ │
│                      │  │ • Confidence score bar           │ │
│                      │  └──────────────────────────────────┘ │
├──────────────────────┴──────────────────────────────────────┤
│  FOOTER: "Built by Nishanth | UMD CPSE '26 | Powered by    │
│  Claude API + MediaPipe"                                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. CORE FEATURES — DETAILED SPECIFICATIONS

### 4.1 — The Grid World

- 30x30 grid rendered on an HTML5 `<canvas>` element (better perf than DOM cells) OR as a CSS grid of divs if simpler. Canvas preferred.
- Each cell can be: EMPTY, OBSTACLE, ROBOT (start), or GOAL (end)
- **Mouse interaction modes** (toggle buttons):
  - **Draw mode**: Click/drag to paint obstacles
  - **Erase mode**: Click/drag to remove obstacles
  - **Place Robot mode**: Click to place the cyan robot
  - **Place Goal mode**: Click to place the amber goal
- Obstacles should feel "solid" — filled cells with a subtle inner shadow
- When hovering over a cell, show a ghost preview of what will be placed
- **Drag-to-replan**: After a path is computed, the user can click-drag any obstacle to a new position. On mouse-up, the pathfinding re-runs automatically AND a new AI explanation is fetched (debounced to 1.5 seconds so we don't spam the API)

### 4.2 — Pathfinding Algorithms (implement from scratch, do NOT use a library)

**A* (A-Star):**
- Standard grid-based A* with 8-directional movement (diagonals allowed)
- Heuristic: Octile distance (proper for 8-dir grids)
- Respect robot size: if robot size is 3, the robot occupies a 3x3 area, so pathfinding must ensure ALL cells in the robot's footprint are clear
- Track and expose: path length, nodes explored, computation time (ms)
- During visualization, animate the exploration (show explored nodes expanding outward in a wave, then the final path draws in — use requestAnimationFrame with a step delay for the animation)

**RRT (Rapidly-exploring Random Tree):**
- Basic RRT with goal biasing (10% chance of sampling the goal directly)
- Step size: 2 cells
- Max iterations: 3000
- Post-process: path smoothing (greedy shortcutting — try connecting non-adjacent nodes with straight lines, keep if collision-free)
- Track and expose: path length, tree size (total nodes), iterations used, computation time
- During visualization, animate the tree growing (draw each new branch as it's added, then highlight the final path)

**Both mode**: Run both algorithms, draw both paths (A* = cyan solid, RRT = amber dashed), show comparison metrics in the Recharts bar chart

### 4.3 — Robot Parameters (Sliders)

- **Robot Size** (1–5 cells): Affects pathfinding collision checking. Visually, the robot icon scales. Label shows current value.
- **Speed** (1–10): Purely visual — affects how fast the path-drawing animation plays. 1 = slow educational mode, 10 = instant.
- **Sensor Range** (1–15 cells): Draws a translucent cyan circle around the robot showing its "sensor footprint." This is sent to Claude API as context so the AI can comment on whether the sensor range is sufficient for the environment's obstacle density.

All sliders should have the "Control Room" style: thin track, cyan fill, no default browser styling.

### 4.4 — AI Tutor Panel (Claude API Integration)

When the user clicks "PLAN PATH" or drags an obstacle (debounced), send a request to Claude API with this payload:

```
System prompt: "You are PathMind AI, a robotics tutor inside a path planning visualizer. You explain path planning algorithms to engineers and recruiters in clear, engaging language. Be concise (4-6 sentences max). Use analogies when helpful. Always mention specific metrics from the data provided. If both algorithms were run, compare them and declare a winner for this specific scenario with reasoning."

User message: Construct from current state:
- Grid dimensions: 30x30
- Obstacle count: {n} 
- Obstacle density: {n / 900 * 100}%
- Obstacle pattern description: {clustered / scattered / walls / maze-like} — compute this heuristically by checking adjacency
- Robot position: ({x}, {y})
- Goal position: ({x}, {y})
- Manhattan distance (start to goal): {d}
- Robot size: {s} cells
- Sensor range: {r} cells
- Algorithm(s) used: {A* / RRT / Both}
- A* results (if run): path length = {L}, nodes explored = {N}, time = {T}ms
- RRT results (if run): path length = {L}, tree nodes = {N}, iterations = {I}, time = {T}ms
- Is the sensor range sufficient to see the nearest obstacle from the start position? {yes/no}

Explain why the algorithm(s) chose the path(s) they did, what the metrics reveal about the environment's complexity, and any robotics insight about this scenario.
```

Display the AI response in the tutor panel with a **typewriter animation** (reveal one character at a time, 20ms interval, with a blinking cursor).

Show a loading state: "PathMind AI is analyzing..." with a pulsing cyan dot animation.

### 4.5 — MediaPipe Hand Tracking Integration

This is the showstopper feature. Load MediaPipe Hands via CDN:

```javascript
// These are the CDN scripts to load dynamically
// @mediapipe/hands
// @mediapipe/camera_utils  
// @mediapipe/drawing_utils
```

**Implementation:**
1. Add a "Hand Control" toggle button in the header. When enabled:
   - Request webcam access (`navigator.mediaDevices.getUserMedia`)
   - Show a small (200x150) webcam preview in the Hand Tracking Status panel
   - Initialize MediaPipe Hands with `maxNumHands: 1`, `minDetectionConfidence: 0.7`
   - Draw hand landmarks on the preview using `drawConnectors` and `drawLandmarks`

2. **Gesture Recognition** (implement these gestures by analyzing landmark positions):

   | Gesture | Detection Logic | Action |
   |---------|----------------|--------|
   | **Index finger pointing** | Only index finger extended (tip above PIP), others curled | Move cursor on grid (map index fingertip x,y from camera space to grid space) |
   | **Pinch (index + thumb)** | Distance between index tip (8) and thumb tip (4) < 40px | Place/toggle obstacle at current cursor position |
   | **Open palm (all fingers spread)** | All 5 fingertips above their respective PIP joints | Clear the grid |
   | **Fist (all fingers curled)** | All fingertips below their PIP joints | Trigger "PLAN PATH" |
   | **Peace sign (index + middle)** | Index and middle extended, others curled | Toggle between Draw/Erase mode |

3. **Gesture state display**: Show the currently detected gesture name and a confidence bar (0-100%) in the Hand Tracking Status panel

4. **Visual cursor**: When hand tracking is active, show a large glowing cyan crosshair on the grid following the index fingertip position. This makes it clear where the "hand cursor" is pointing.

5. **Mirror the webcam** horizontally so it feels natural (CSS `transform: scaleX(-1)`)

**IMPORTANT**: MediaPipe may not load in the artifact sandbox. That's OK. Implement it fully, but add graceful error handling: if MediaPipe fails to load, show a message "Hand tracking unavailable in this environment — works when deployed locally" and keep all mouse controls working perfectly. The code being there (and visible on GitHub) is what matters.

---

## 5. ANIMATIONS & VISUAL POLISH

- **Grid load-in**: Cells fade in with a staggered wave animation on first render (top-left to bottom-right, 2ms delay per cell)
- **Path drawing**: After computation, animate the path being drawn cell by cell (like a snake). A* path = cyan glow trail, RRT = amber dashed trail
- **Exploration visualization**: Show A*'s open set expanding as a wave of semi-transparent cyan, and RRT's tree branches growing outward
- **Robot idle animation**: Subtle pulsing glow + gentle rotation of the directional triangle
- **Goal animation**: Slow continuous rotation (CSS `@keyframes rotate`)
- **Button hover states**: Cyan border glow, slight scale(1.02)
- **Panel transitions**: Slide-in from right when metrics/AI response loads
- **Scan-line overlay on the entire app**: `repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0,229,255,0.03) 2px, rgba(0,229,255,0.03) 4px)` — very subtle, gives it the "monitor" feel

---

## 6. EDGE CASES & ERROR HANDLING

- **No path exists**: If the goal is unreachable (fully walled off), display "NO PATH FOUND" in red on the grid, and have the AI tutor explain WHY no path exists and suggest what the user could change
- **Robot/goal overlap**: Prevent placing both on the same cell
- **Empty grid**: If user clicks PLAN PATH with no obstacles, still run the algorithms (straight line) and have the AI comment on trivial environments
- **API failure**: If the Claude API call fails, show "AI tutor offline — check connection" and still display the algorithmic results
- **MediaPipe failure**: Graceful fallback (see section 4.5)
- **Performance**: Debounce drag-to-replan at 1500ms. Don't re-run AI on every frame.

---

## 7. PRESET SCENARIOS (dropdown or buttons)

Include 4-5 preset obstacle layouts so recruiters don't have to draw from scratch:

1. **"Maze"** — A simple maze with corridors
2. **"Scattered"** — Random sparse obstacles (15% density)
3. **"Wall with gap"** — A wall across the middle with one small gap
4. **"Cluttered room"** — Dense random obstacles (40% density)
5. **"U-Trap"** — A U-shaped obstacle that forces backtracking (great for showing A* vs RRT differences)

Each preset should also set the robot and goal in interesting positions.

---

## 8. METRICS VISUALIZATION (Recharts)

Below the control panel, render a small Recharts `<BarChart>` comparing A* vs RRT when both are run:

- Bars for: Path Length, Nodes Explored, Computation Time (ms)
- Use cyan for A* bars, amber for RRT bars
- Animate bars on load
- Dark background matching the app theme
- Clean axis labels in JetBrains Mono

---

## 9. CODE QUALITY REQUIREMENTS

- Clean, well-commented code. Every major function has a JSDoc-style comment explaining what it does
- Organize code into clear sections with banner comments:
  ```
  // ═══════════════════════════════════════════
  //  SECTION: A* PATHFINDING
  // ═══════════════════════════════════════════
  ```
- Use meaningful variable names (not `x1`, `tmp`, etc.)
- Extract magic numbers into named constants at the top:
  ```javascript
  const GRID_SIZE = 30;
  const CELL_SIZE = 20;
  const RRT_MAX_ITERATIONS = 3000;
  const RRT_STEP_SIZE = 2;
  const RRT_GOAL_BIAS = 0.1;
  const AI_DEBOUNCE_MS = 1500;
  ```

---

## 10. GITHUB REPOSITORY STRUCTURE

After building the artifact, also generate these files for my GitHub repo:

### 10.1 — README.md
A polished, LinkedIn-worthy README with:
- Project banner description (use text, not an image)
- "What is PathMind?" section
- Feature list with descriptions
- Tech stack section
- How the AI integration works (brief architecture explanation)
- How hand tracking works (brief explanation with gesture table)
- "How to Run" section (just open the deployed link or paste into Claude artifact)
- Algorithm explanations (A* and RRT, 2-3 sentences each)
- Screenshots section (placeholder text — I'll add these)
- "Built By" section with my name, degree, university
- License: MIT

### 10.2 — ARCHITECTURE.md
A technical deep-dive document explaining:
- System architecture (client-only, no backend)
- Data flow: user input → grid state → algorithm execution → AI analysis → display
- Algorithm implementation details (A* heuristic choice, RRT parameters)
- MediaPipe integration architecture
- Claude API prompt engineering approach
- Performance considerations and debouncing strategy

### 10.3 — LICENSE
Standard MIT license

### 10.4 — .gitignore
Standard JS gitignore

---

## 11. WHAT TO BUILD (execution order)

1. **First**: Build the complete React artifact (.jsx) with ALL features — grid, pathfinding, AI tutor, MediaPipe, animations, presets, metrics chart. This is the main deliverable.
2. **Second**: Generate the README.md file
3. **Third**: Generate the ARCHITECTURE.md file
4. **Fourth**: Generate LICENSE and .gitignore

---

## REMEMBER

- This is a PORTFOLIO PROJECT for a CPS engineer. Every detail matters.
- The code must be REAL and FUNCTIONAL, not a mockup
- The A* and RRT algorithms must be CORRECTLY IMPLEMENTED from scratch
- The Claude API integration must actually call the API and display real responses
- The MediaPipe code must be real (even if it can't run in the sandbox)
- The UI must look like it was designed by someone who cares, not generated by default AI
- Comments in code should show I understand the robotics concepts, not just the code
- This needs to impress a hiring manager at a robotics/defense/IoT company who clicks the link on my LinkedIn

Build it all now. Start with the main React artifact.

---

## PROMPT END


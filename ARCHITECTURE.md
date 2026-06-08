# PathMind — Architecture Document

## System Architecture

PathMind is a **fully client-side** single-page application. There is no backend server, no database, and no build step. The entire application runs in a single `index.html` file that loads its dependencies (React, Recharts, MediaPipe, Lucide) via CDN.

```
┌──────────────────────────────────────────────────────────────┐
│                        BROWSER                               │
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                   index.html                             │ │
│  │                                                          │ │
│  │  ┌──────────┐  ┌────────────┐  ┌───────────────────┐    │ │
│  │  │  React   │  │  Recharts  │  │   Lucide Icons    │    │ │
│  │  │  18 CDN  │  │    CDN     │  │      CDN          │    │ │
│  │  └──────────┘  └────────────┘  └───────────────────┘    │ │
│  │                                                          │ │
│  │  ┌──────────────────────────────────────────────────┐   │ │
│  │  │              APPLICATION LAYER                    │   │ │
│  │  │                                                    │   │ │
│  │  │  Grid State ──► Algorithms ──► Animation Engine    │   │ │
│  │  │      │              │                │             │   │ │
│  │  │      │              ▼                ▼             │   │ │
│  │  │      │        Result Metrics    Canvas Renderer    │   │ │
│  │  │      │              │                              │   │ │
│  │  │      ▼              ▼                              │   │ │
│  │  │  Claude API    Recharts Panel                      │   │ │
│  │  │  (External)                                        │   │ │
│  │  └──────────────────────────────────────────────────┘   │ │
│  │                                                          │ │
│  │  ┌──────────────────────────────────────────────────┐   │ │
│  │  │           MEDIAPIPE LAYER (Optional)              │   │ │
│  │  │                                                    │   │ │
│  │  │  Webcam ──► MediaPipe Hands ──► Gesture Engine    │   │ │
│  │  │                                     │              │   │ │
│  │  │                                     ▼              │   │ │
│  │  │                              Grid Interaction      │   │ │
│  │  └──────────────────────────────────────────────────┘   │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
│  External API Call ─────────────────────────────────────────►│
│                          Claude Sonnet API                    │
└──────────────────────────────────────────────────────────────┘
```

---

## Data Flow

```
User Input (mouse/hand gesture)
    │
    ▼
Grid State Update (30x30 2D array: 0=empty, 1=obstacle)
    │
    ├──► Robot position [x, y]
    ├──► Goal position [x, y]
    └──► Robot parameters (size, speed, sensor range)
         │
         ▼
    Algorithm Execution (triggered by "Plan Path" or drag-to-replan)
         │
         ├──► A* Search ──► { path, explored[], nodesExplored, timeMs }
         └──► RRT Search ──► { path, treeEdges[], treeSize, iterations, timeMs }
              │
              ├──► Animation Engine (requestAnimationFrame step-by-step reveal)
              │         │
              │         └──► Canvas Renderer (draws grid, paths, robot, goal per frame)
              │
              ├──► Recharts Metrics (bar chart comparing A* vs RRT)
              │
              └──► Claude API Request
                        │
                        ▼
                   AI Tutor Response (typewriter animation in panel)
```

---

## Algorithm Implementation Details

### A\* (A-Star)

**Heuristic:** Octile distance — the optimal admissible heuristic for 8-directional grid movement:

```
h(n) = max(dx, dy) + (√2 - 1) * min(dx, dy)
```

This is tighter than Manhattan distance (which overestimates for diagonal movement) and ensures A\* finds the true shortest path on an 8-connected grid.

**Movement:** 8-directional (cardinal + diagonal). Cardinal moves cost 1.0, diagonal moves cost √2 ≈ 1.414.

**Robot size handling:** When `robotSize > 1`, the robot occupies an `N×N` footprint. The `isRobotClear()` function checks that ALL cells in the footprint are obstacle-free before allowing the robot to occupy that position. This effectively inflates obstacles by the robot's radius.

**Data structures:** Uses a sorted array as the priority queue. For a 30×30 grid (900 cells max), this is efficient enough — a binary heap would be over-engineered for this scale.

**Complexity:** O(V log V) where V = accessible cells ≤ 900. In practice, runs in <5ms for most scenarios.

### RRT (Rapidly-exploring Random Tree)

**Sampling strategy:** Uniform random sampling with 10% goal bias. The goal bias prevents the tree from wandering aimlessly in open environments while still maintaining the probabilistic completeness guarantees of RRT.

**Step size:** 2 cells per extension. This balances between fine-grained exploration (step=1, slow convergence) and missing narrow passages (step>3, collision misses).

**Collision checking:** The `lineOfSight()` function samples points along the line segment between two nodes at sub-cell resolution (2× the Euclidean distance in sample points). Each sample point is checked against the inflated obstacle map.

**Path smoothing:** After finding a raw path, greedy shortcutting iterates through waypoints and attempts to connect non-adjacent nodes with straight lines. If the line is collision-free, intermediate waypoints are removed. This typically reduces path length by 30-50%.

**Max iterations:** 3000. For a 30×30 grid, this provides high probability of finding a path if one exists. The algorithm terminates early upon reaching the goal.

---

## MediaPipe Integration Architecture

MediaPipe Hands is loaded dynamically via CDN scripts to avoid blocking initial page render. The initialization sequence:

1. User enables hand tracking toggle
2. Three CDN scripts load sequentially: `hands.js`, `camera_utils.js`, `drawing_utils.js`
3. Webcam access is requested via `getUserMedia`
4. A `Camera` utility feeds video frames to the `Hands` model
5. On each frame result, the app:
   - Draws hand landmarks on a preview canvas (mirrored via CSS)
   - Classifies the gesture by analyzing landmark geometry
   - Maps the index fingertip position to grid coordinates
   - Executes the corresponding grid action (debounced at 800ms for non-cursor gestures)

**Gesture detection** is purely geometric — no ML classifier beyond MediaPipe's own hand detection. Finger extension is determined by comparing the Y-coordinate of each fingertip landmark (indices 4, 8, 12, 16, 20) against its PIP joint (indices 3, 6, 10, 14, 18). The pinch gesture uses Euclidean distance between landmarks 4 and 8.

**Graceful degradation:** If MediaPipe CDN scripts fail to load (common in sandboxed environments), the app catches the error and displays a fallback message. All mouse-based controls remain fully functional.

---

## Claude API Prompt Engineering

The AI tutor uses a carefully structured prompt:

**System prompt** establishes the persona (robotics tutor), tone (clear, engaging, concise), and output constraints (4-6 sentences, use metrics, declare a winner when comparing).

**User message** is dynamically constructed from the current grid state and algorithm results. Key context includes:
- Quantitative metrics (path length, nodes, time)
- Qualitative descriptors (obstacle pattern classification via adjacency analysis)
- Robot configuration (size and sensor range)
- Environmental context (sensor sufficiency check)

The obstacle pattern classifier uses average neighbor adjacency per obstacle cell:
- `> 2.5` neighbors → maze-like (high connectivity = corridors)
- `> 1.5` → walls (linear structures)
- `> 0.8` → clustered (grouped but not linear)
- `≤ 0.8` → scattered (isolated obstacles)

This gives the AI enough context to provide genuinely insightful analysis rather than generic descriptions.

---

## Performance Considerations

### Rendering
- HTML5 Canvas is used instead of DOM elements for the grid (900 cells × 60fps would create excessive DOM overhead)
- A single `requestAnimationFrame` loop handles all rendering, including robot pulse and goal rotation animations
- The canvas is 600×600px (30 cells × 20px), keeping draw calls fast

### Algorithm execution
- Pathfinding runs in a `setTimeout(..., 50)` to yield to the UI thread before heavy computation
- A\* and RRT are synchronous but complete in <50ms for typical 30×30 scenarios

### Debouncing
- Drag-to-replan is debounced at 1500ms to prevent API spam during continuous obstacle dragging
- Hand gesture actions (except cursor movement) are debounced at 800ms to prevent repeated triggers from sustained gestures
- The typewriter effect uses a 15ms interval for smooth character reveal

### Memory
- Grid state is a flat 30×30 array (900 integers)
- Animation state stores coordinate arrays that are replaced (not accumulated) on each new pathfinding run
- No persistent memory leaks — animation frames and intervals are cleaned up on component re-render

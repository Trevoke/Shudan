# Shudan

A Go board component for the web. Render beautiful, interactive game boards with stones, markers, territory visualization, and AI move suggestions.

**For:** Developers building Go/Weiqi/Baduk applications, game editors, teaching tools, or AI analysis interfaces.

**Powers:** [Sabaki](https://sabaki.yichuanshen.de), the popular open-source Go game editor.

---

## Your First Board in 60 Seconds

```jsx
import { Goban } from "@sabaki/shudan";
import "@sabaki/shudan/css/goban.css";

// A 9x9 board with a few stones
const signMap = [
  [0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 1, 0, 0, 0, -1, 0, 0],  // 1 = black, -1 = white, 0 = empty
  [0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, -1, 0, 0, 0, 1, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0],
  [0, 0, 0, 0, 0, 0, 0, 0, 0],
];

function App() {
  return <Goban vertexSize={24} signMap={signMap} />;
}
```

That's it. You have a working Go board.

### Installation

```bash
npm install @sabaki/shudan
```

Include the CSS (required for board and stone rendering):

```html
<link rel="stylesheet" href="node_modules/@sabaki/shudan/css/goban.css" />
```

Or import it in your bundler:

```js
import "@sabaki/shudan/css/goban.css";
```

### Using with React

Shudan is written for Preact but works with React. Add this to your bundler config:

```js
// webpack.config.js
module.exports = {
  resolve: {
	alias: {
	  preact: "react",
	  "preact/hooks": "react",
	},
  },
};
```

---

## Understanding the Board

### How Shudan Represents a Go Board

Think of a Go board as a grid of intersections. Shudan represents this grid as a 2D array where:

- Each row is an array
- Each element represents one intersection
- Values: `1` (black stone), `-1` (white stone), `0` (empty)

```js
// A simple 3x3 corner:
[
  [1, 0, -1],   // Row 0: black at (0,0), empty at (1,0), white at (2,0)
  [0, 1, 0],    // Row 1: black at (1,1)
  [0, 0, 0],    // Row 2: all empty
]
```

This structure is called a **map** in Shudan. You'll use maps for stones, markers, heat visualization, and more.

### Coordinates and Positions

Positions on the board are called **vertices**. A vertex is an `[x, y]` pair:

- `x` is the column (0 = leftmost)
- `y` is the row (0 = topmost)
- `[0, 0]` is the **top-left corner**

```
	   x=0  x=1  x=2
		|    |    |
y=0 --- A    B    C
y=1 --- D    E    F
y=2 --- G    H    I

Vertex [1, 2] = position H
```

---

## Tutorial: Building an Interactive Board

Let's build a board where users can place stones by clicking.

### Step 1: Track Board State

```jsx
import { useState } from "preact/hooks"; // or "react"
import { Goban } from "@sabaki/shudan";

function InteractiveBoard() {
  // Create a 9x9 empty board
  const [signMap, setSignMap] = useState(
	Array(9).fill(null).map(() => Array(9).fill(0))
  );
  const [currentPlayer, setCurrentPlayer] = useState(1); // 1 = black's turn

  return (
	<Goban
	  vertexSize={24}
	  signMap={signMap}
	/>
  );
}
```

### Step 2: Handle Clicks

Add the `onVertexClick` handler to respond when users click intersections:

```jsx
function InteractiveBoard() {
  const [signMap, setSignMap] = useState(
	Array(9).fill(null).map(() => Array(9).fill(0))
  );
  const [currentPlayer, setCurrentPlayer] = useState(1);

  const handleClick = (evt, [x, y]) => {
	// Only place on empty intersections
	if (signMap[y][x] !== 0) return;

	// Create new board state with the stone placed
	const newSignMap = signMap.map((row, rowIndex) =>
	  row.map((cell, colIndex) =>
		rowIndex === y && colIndex === x ? currentPlayer : cell
	  )
	);

	setSignMap(newSignMap);
	setCurrentPlayer(currentPlayer * -1); // Switch players
  };

  return (
	<Goban
	  vertexSize={24}
	  signMap={signMap}
	  onVertexClick={handleClick}
	/>
  );
}
```

### Step 3: Show Whose Turn It Is

Use ghost stones to preview where the next stone will be placed:

```jsx
function InteractiveBoard() {
  const [signMap, setSignMap] = useState(
	Array(9).fill(null).map(() => Array(9).fill(0))
  );
  const [currentPlayer, setCurrentPlayer] = useState(1);
  const [hoverVertex, setHoverVertex] = useState(null);

  // Create ghost stone map showing hover preview
  const ghostStoneMap = signMap.map((row, y) =>
	row.map((cell, x) => {
	  if (hoverVertex && hoverVertex[0] === x && hoverVertex[1] === y && cell === 0) {
		return { sign: currentPlayer, faint: true };
	  }
	  return null;
	})
  );

  return (
	<Goban
	  vertexSize={24}
	  signMap={signMap}
	  ghostStoneMap={ghostStoneMap}
	  onVertexClick={(evt, [x, y]) => {
		if (signMap[y][x] !== 0) return;
		const newSignMap = signMap.map((row, ri) =>
		  row.map((cell, ci) => (ri === y && ci === x ? currentPlayer : cell))
		);
		setSignMap(newSignMap);
		setCurrentPlayer(currentPlayer * -1);
	  }}
	  onVertexMouseEnter={(evt, vertex) => setHoverVertex(vertex)}
	  onVertexMouseLeave={() => setHoverVertex(null)}
	/>
  );
}
```

---

## Tutorial: Showing Game Analysis

Go AI tools like KataGo suggest moves with win rates and visit counts. Here's how to display that information.

### Heat Maps for Move Strength

Heat maps show AI-suggested moves with color-coded intensity (green = best, red = worst):

```jsx
// heatMap is a 2D array matching your board size
const heatMap = signMap.map((row) => row.map(() => null));

// Top suggestions from your AI
const suggestions = [
  { x: 3, y: 3, strength: 9, text: "52.3%\n1.2k" },
  { x: 15, y: 3, strength: 7, text: "48.1%\n800" },
  { x: 3, y: 15, strength: 5, text: "45.2%\n400" },
];

suggestions.forEach(({ x, y, strength, text }) => {
  heatMap[y][x] = { strength, text };
});

<Goban signMap={signMap} heatMap={heatMap} />
```

`strength` ranges from 1-9:
- **9**: Best move (bright green)
- **7-8**: Strong move (blue-green)
- **5-6**: Decent move (blue)
- **3-4**: Weak move (purple)
- **1-2**: Poor move (red)

### Markers for Move Annotations

Add symbols to mark important positions:

```jsx
const markerMap = signMap.map((row) => row.map(() => null));

// Mark the last move with a circle
markerMap[lastMove.y][lastMove.x] = { type: "circle" };

// Mark a sequence with letters
markerMap[3][3] = { type: "label", label: "A" };
markerMap[3][15] = { type: "label", label: "B" };
markerMap[15][3] = { type: "label", label: "C" };

<Goban signMap={signMap} markerMap={markerMap} />
```

Available marker types:
- `circle` - Circle around stone/intersection
- `cross` - X mark
- `triangle` - Triangle
- `square` - Square
- `point` - Small dot
- `label` - Text label (use with `label` property)
- `loader` - Spinning circle (for "thinking" indicators)

### Territory Visualization with Paint Maps

Show territory ownership or influence:

```jsx
// Values from -1 to 1: negative = white territory, positive = black
const paintMap = signMap.map((row) => row.map(() => 0));

// Mark black territory
paintMap[0][0] = 0.8;  // Strongly black
paintMap[0][1] = 0.6;
paintMap[1][0] = 0.4;  // Lighter influence

// Mark white territory
paintMap[18][18] = -0.8;  // Strongly white

<Goban signMap={signMap} paintMap={paintMap} />
```

---

## Tutorial: Reviewing Games

### Showing Move Variations

Ghost stones show alternative moves or variations without placing actual stones:

```jsx
const ghostStoneMap = signMap.map((row) => row.map(() => null));

// Show a good alternative move
ghostStoneMap[4][4] = { sign: 1, type: "good" };

// Show a questionable move
ghostStoneMap[10][10] = { sign: -1, type: "doubtful" };

<Goban signMap={signMap} ghostStoneMap={ghostStoneMap} />
```

Ghost stone types:
- No type: Plain semi-transparent stone
- `good`: Green highlight
- `interesting`: Blue highlight
- `doubtful`: Purple highlight
- `bad`: Red highlight

Add `faint: true` for even more transparency.

### Drawing Lines and Arrows

Connect related positions:

```jsx
<Goban
  signMap={signMap}
  lines={[
	{ v1: [3, 3], v2: [15, 15], type: "line" },
	{ v1: [10, 5], v2: [5, 10], type: "arrow" },
  ]}
/>
```

### Dimming Dead Stones

During scoring, dim captured or dead stones:

```jsx
<Goban
  signMap={signMap}
  dimmedVertices={[
	[5, 5],   // These stones appear faded
	[5, 6],
	[6, 5],
  ]}
/>
```

### Selecting Regions

Highlight a group of positions:

```jsx
<Goban
  signMap={signMap}
  selectedVertices={[
	[10, 10],
	[10, 11],
	[11, 10],
	[11, 11],
  ]}
/>
```

Adjacent selected vertices merge into a single selection box.

---

## Tutorial: Partial Boards and Sizing

### Showing Part of the Board

Focus on a corner or region:

```jsx
// Show only the lower-right corner (columns 13-18, rows 13-18)
<Goban
  signMap={signMap}
  rangeX={[13, 18]}
  rangeY={[13, 18]}
/>
```

### Auto-Sizing with BoundedGoban

When you need the board to fit a container rather than specify exact pixel sizes:

```jsx
import { BoundedGoban } from "@sabaki/shudan";

// Board will scale to fit within 400x400, maintaining aspect ratio
<BoundedGoban
  maxWidth={400}
  maxHeight={400}
  signMap={signMap}
/>
```

Add `maxVertexSize` to prevent vertices from getting too large:

```jsx
<BoundedGoban
  maxWidth={800}
  maxHeight={800}
  maxVertexSize={30}  // Never larger than 30px per intersection
  signMap={signMap}
/>
```

---

## Visual Effects

### Fuzzy Stone Placement

Real Go stones don't sit perfectly centered. Enable natural-looking placement:

```jsx
<Goban
  signMap={signMap}
  fuzzyStonePlacement={true}
/>
```

### Animated Stone Placement

Stones slide into position when placed:

```jsx
<Goban
  signMap={signMap}
  fuzzyStonePlacement={true}   // Required for animation
  animateStonePlacement={true}
/>
```

Note: Animation only triggers when the `signMap` prop is a **new object**. If you mutate the existing array, no animation occurs.

### Busy State

Show a loading overlay while processing:

```jsx
<Goban
  signMap={signMap}
  busy={true}
/>
```

When `busy` is true, a shimmer animation plays over the board and clicks are ignored.

---

## Customizing Appearance

### Board Coordinates

Show column letters and row numbers:

```jsx
<Goban
  signMap={signMap}
  showCoordinates={true}
/>
```

Customize the coordinate labels:

```jsx
<Goban
  signMap={signMap}
  showCoordinates={true}
  coordX={(i) => String.fromCharCode(65 + i)}  // A, B, C, ...
  coordY={(i) => 19 - i}                        // 19, 18, 17, ...
/>
```

For Chinese numerals or other systems, provide your own mapping functions.

### CSS Customization

Override CSS custom properties for colors:

```css
.shudan-goban {
  --shudan-board-border-width: 0.25em;
  --shudan-board-border-color: #ca933a;

  --shudan-board-background-color: #ebb55b;
  --shudan-board-foreground-color: #5e2e0c;

  --shudan-black-background-color: #222;
  --shudan-black-foreground-color: #eee;

  --shudan-white-background-color: #fff;
  --shudan-white-foreground-color: #222;
}
```

### Custom Stone Images

Replace default stones with your own images:

```css
.shudan-stone-image.shudan-sign_1 {
  background-image: url("./my-black-stone.png");
}

.shudan-stone-image.shudan-sign_-1 {
  background-image: url("./my-white-stone.png");
}
```

### Stone Variety with Random Classes

Shudan adds `.shudan-random_0` through `.shudan-random_4` to each stone. Use these for shell pattern variety:

```css
.shudan-stone-image.shudan-sign_-1 {
  background-image: url("white_shell_1.png");
}
.shudan-stone-image.shudan-sign_-1.shudan-random_1 {
  background-image: url("white_shell_2.png");
}
.shudan-stone-image.shudan-sign_-1.shudan-random_2 {
  background-image: url("white_shell_3.png");
}
/* ... and so on */
```

### Custom Board Texture

```css
.shudan-goban-image {
  background-image: url("./my-wood-texture.png");
}
```

---

## API Reference

### Goban Component

The main board component. All props are optional.

#### Sizing

| Prop         | Type               | Default         | Description                         |
|--------------+--------------------+-----------------+-------------------------------------|
| `vertexSize` | `number`           | `24`            | Size of each intersection in pixels |
| `rangeX`     | `[number, number]` | `[0, Infinity]` | Visible column range (inclusive)    |
| `rangeY`     | `[number, number]` | `[0, Infinity]` | Visible row range (inclusive)       |

#### Board State

| Prop      | Type         | Default | Description                                       |
|-----------+--------------+---------+---------------------------------------------------|
| `signMap` | `number[][]` | `[]`    | Stone positions: `1` black, `-1` white, `0` empty |
| `busy`    | `boolean`    | `false` | Shows loading overlay, disables interaction       |

#### Visual Effects

| Prop                    | Type                    | Default      | Description                                         |
|-------------------------+-------------------------+--------------+-----------------------------------------------------|
| `showCoordinates`       | `boolean`               | `false`      | Display column/row labels                           |
| `coordX`                | `(x: number) => string` | A-Z (skip I) | Column label function                               |
| `coordY`                | `(y: number) => string` | height - y   | Row label function                                  |
| `fuzzyStonePlacement`   | `boolean`               | `false`      | Stones slightly off-grid                            |
| `animateStonePlacement` | `boolean`               | `false`      | Animate new stones (requires `fuzzyStonePlacement`) |

#### Overlay Maps

| Prop            | Type             | Description                     |
|-----------------+------------------+---------------------------------|
| `markerMap`     | `Marker[][]`     | Markers on intersections        |
| `ghostStoneMap` | `GhostStone[][]` | Semi-transparent preview stones |
| `paintMap`      | `number[][]`     | Territory shading (-1 to 1)     |
| `heatMap`       | `HeatVertex[][]` | AI suggestion visualization     |

#### Vertex State

| Prop               | Type                 | Default | Description                    |
|--------------------+----------------------+---------+--------------------------------|
| `selectedVertices` | `[number, number][]` | `[]`    | Highlighted positions          |
| `dimmedVertices`   | `[number, number][]` | `[]`    | Faded positions (dead stones)  |
| `lines`            | `LineMarker[]`       | `[]`    | Lines/arrows between positions |

#### Events

All event handlers receive `(event, vertex)` where `vertex` is `[x, y]`:

| Prop                   | Event                         |
|------------------------+-------------------------------|
| `onVertexClick`        | Click on intersection         |
| `onVertexMouseDown`    | Mouse button pressed          |
| `onVertexMouseUp`      | Mouse button released         |
| `onVertexMouseMove`    | Mouse moved over intersection |
| `onVertexMouseEnter`   | Mouse entered intersection    |
| `onVertexMouseLeave`   | Mouse left intersection       |
| `onVertexPointerDown`  | Pointer pressed (touch/mouse) |
| `onVertexPointerUp`    | Pointer released              |
| `onVertexPointerMove`  | Pointer moved                 |
| `onVertexPointerEnter` | Pointer entered               |
| `onVertexPointerLeave` | Pointer left                  |

#### DOM Props

| Prop                  | Type     | Description                            |
|-----------------------+----------+----------------------------------------|
| `id`                  | `string` | Sets container `id` attribute          |
| `class` / `className` | `string` | Additional CSS classes                 |
| `style`               | `object` | Inline styles                          |
| `innerProps`          | `object` | Additional props for container element |

---

### BoundedGoban Component

Automatically sizes the board to fit within maximum dimensions. Inherits all `Goban` props except `vertexSize`.

| Prop            | Type         | Default    | Description                   |
|-----------------+--------------+------------+-------------------------------|
| `maxWidth`      | `number`     | required   | Maximum width in pixels       |
| `maxHeight`     | `number`     | required   | Maximum height in pixels      |
| `maxVertexSize` | `number`     | `Infinity` | Cap on intersection size      |
| `onResized`     | `() => void` | -          | Called after resize completes |

---

### Type Definitions

#### Marker

```typescript
interface Marker {
  type?: "circle" | "cross" | "triangle" | "square" | "point" | "loader" | "label" | null;
  label?: string | null;  // Text for label type, also used as tooltip
}
```

#### GhostStone

```typescript
interface GhostStone {
  sign: 0 | -1 | 1;  // Which color stone to show
  type?: "good" | "interesting" | "doubtful" | "bad" | null;
  faint?: boolean | null;  // Extra transparency
}
```

#### HeatVertex

```typescript
interface HeatVertex {
  strength: number;  // 1-9, affects color/size
  text?: string | null;  // Displayed in center
}
```

#### LineMarker

```typescript
interface LineMarker {
  v1: [number, number];  // Start vertex
  v2: [number, number];  // End vertex
  type?: "line" | "arrow";
}
```

---

### CSS Classes

All Shudan classes are prefixed with `shudan-`:

| Class                               | Element                 |
|-------------------------------------+-------------------------|
| `.shudan-goban`                     | Main container          |
| `.shudan-goban-image`               | Board background        |
| `.shudan-vertex`                    | Individual intersection |
| `.shudan-stone`                     | Stone container         |
| `.shudan-stone-image`               | Stone graphic           |
| `.shudan-marker`                    | Marker symbols          |
| `.shudan-ghost`                     | Ghost stone             |
| `.shudan-heat`                      | Heat map glow           |
| `.shudan-heatlabel`                 | Heat map text           |
| `.shudan-paint`                     | Territory shading       |
| `.shudan-selection`                 | Selection highlight     |
| `.shudan-grid`                      | Grid lines              |
| `.shudan-hoshi`                     | Star points             |
| `.shudan-coordx` / `.shudan-coordy` | Coordinate labels       |
| `.shudan-line` / `.shudan-arrow`    | Connecting lines        |

State modifiers on `.shudan-vertex`:
- `.shudan-sign_1` / `.shudan-sign_-1` / `.shudan-sign_0` - Stone color
- `.shudan-random_0` through `.shudan-random_4` - Randomization for variety
- `.shudan-dimmed` - Faded state
- `.shudan-selected` - Selected state
- `.shudan-busy` - Loading state (on `.shudan-goban`)

---

## License

MIT License. Created by Yichuan Shen.

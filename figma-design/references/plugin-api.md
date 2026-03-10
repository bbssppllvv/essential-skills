# Figma Plugin API Reference for AI Agents

## Table of Contents
1. [Architecture](#1-architecture)
2. [Node Creation](#2-node-creation)
3. [Node Types (37+)](#3-node-types)
4. [Universal Node Properties](#4-universal-node-properties)
5. [Fills, Strokes, Effects](#5-fills-strokes-effects)
6. [Text Manipulation](#6-text-manipulation)
7. [Auto Layout](#7-auto-layout)
8. [Variables and Design Tokens](#8-variables-and-design-tokens)
9. [Components and Instances](#9-components-and-instances)
10. [Named Styles](#10-named-styles)
11. [Vector Editing](#11-vector-editing)
12. [Prototyping](#12-prototyping)
13. [Export](#13-export)
14. [Events](#14-events)
15. [Limitations](#15-limitations)

---

## 1. Architecture

**Dual-thread sandbox model:**
- **Main thread** — Full read/write to document scene. No browser APIs (no DOM, no fetch).
- **UI thread** (iframe) — Full browser APIs. No direct document access.
- Communication via `postMessage` / `onmessage` (JSON-serializable data only).

Only one plugin runs at a time. No background execution. User-initiated only.

---

## 2. Node Creation

### Basic Shapes
```
figma.createRectangle()    → RectangleNode
figma.createEllipse()      → EllipseNode
figma.createPolygon()      → PolygonNode
figma.createStar()         → StarNode
figma.createLine()         → LineNode
figma.createVector()       → VectorNode
```

### Text
```
figma.createText()                              → TextNode
figma.createTextPath(node, startSeg, startPos)  → TextPathNode
```

### Containers
```
figma.createFrame()              → FrameNode
figma.createComponent()          → ComponentNode (Design only)
figma.createComponentFromNode()  → ComponentNode (Design only)
figma.createPage()               → PageNode (Design only)
figma.createSection()            → SectionNode
figma.createSlice()              → SliceNode
```

### Grouping & Boolean Operations
```
figma.group(nodes, parent)       → GroupNode
figma.ungroup(node)              → SceneNode[]
figma.union(nodes, parent)       → BooleanOperationNode
figma.subtract(nodes, parent)    → BooleanOperationNode
figma.intersect(nodes, parent)   → BooleanOperationNode
figma.exclude(nodes, parent)     → BooleanOperationNode
figma.flatten(nodes)             → VectorNode
figma.combineAsVariants(components, parent) → ComponentSetNode
```

### Media & Content
```
figma.createImage(data: Uint8Array)      → Image
figma.createImageAsync(src: string)      → Promise<Image>
figma.createVideoAsync(data: Uint8Array) → Promise<Video>
figma.createNodeFromSvg(svg: string)     → FrameNode
figma.createNodeFromJSXAsync(jsx)        → Promise<SceneNode>
```

### FigJam Only
```
figma.createSticky()          → StickyNode
figma.createShapeWithText()   → ShapeWithTextNode
figma.createConnector()       → ConnectorNode
figma.createCodeBlock()       → CodeBlockNode
figma.createTable(rows, cols) → TableNode
```

---

## 3. Node Types

| Type | Context | Description |
|---|---|---|
| DocumentNode | All | Root of document |
| PageNode | All | Individual pages |
| FrameNode | All | Primary container, supports auto layout |
| GroupNode | All | Simple grouping |
| SectionNode | All | Organization container |
| RectangleNode | All | Rectangle |
| EllipseNode | All | Ellipse/circle |
| PolygonNode | All | Regular polygon |
| StarNode | All | Star shape |
| LineNode | All | Line |
| VectorNode | All | Custom vector paths |
| TextNode | All | Text content |
| TextPathNode | All | Text on a path |
| ComponentNode | Design | Reusable component definition |
| ComponentSetNode | Design | Variant set container |
| InstanceNode | All | Component instance |
| BooleanOperationNode | All | Boolean op result |
| SliceNode | All | Export region |
| StickyNode | FigJam | Sticky note |
| ConnectorNode | FigJam | Connector line |
| TableNode | FigJam | Table |
| WidgetNode | All | Widget instance |
| MediaNode | FigJam | Media element |

---

## 4. Universal Node Properties

### Identity
- `id` (readonly) — unique identifier
- `name` — display name
- `type` (readonly) — node type discriminator
- `parent` (readonly), `removed` (readonly)
- `locked`, `visible`, `expanded`

### Transform & Position
- `x`, `y` — position relative to parent
- `width`, `height` — dimensions
- `rotation` — angle in degrees
- `absoluteTransform`, `relativeTransform` — transform matrices
- `absoluteBoundingBox`, `absoluteRenderBounds` — bounding rects

### Sizing & Constraints
- `constraints: { horizontal, vertical }` — MIN, MAX, CENTER, STRETCH, SCALE
- `resize(w, h)` — resize respecting constraints
- `resizeWithoutConstraints(w, h)` — resize ignoring constraints
- `minWidth`, `maxWidth`, `minHeight`, `maxHeight`

### Visual
- `fills: Paint[]` — fill layers
- `strokes: Paint[]` — stroke layers
- `strokeWeight` — thickness
- `strokeAlign` — INSIDE, OUTSIDE, CENTER
- `effects: Effect[]` — shadows, blurs
- `opacity` — 0 to 1
- `blendMode` — NORMAL, MULTIPLY, SCREEN, etc.
- `cornerRadius` — uniform corners
- `topLeftRadius`, `topRightRadius`, `bottomLeftRadius`, `bottomRightRadius` — individual corners
- `cornerSmoothing` — 0 to 1 (iOS-style smooth)

### Operations
- `node.remove()` — delete node
- `node.exportAsync(settings?)` — export as image
- `node.getCSSAsync()` — get CSS properties
- `node.findAll(callback)` — recursive search
- `node.findOne(callback)` — first match

---

## 5. Fills, Strokes, Effects

### Paint Types

**SolidPaint**: `{ type: 'SOLID', color: {r, g, b}, opacity?, visible?, blendMode? }`

**GradientPaint**: `{ type: 'GRADIENT_LINEAR' | 'GRADIENT_RADIAL' | 'GRADIENT_ANGULAR' | 'GRADIENT_DIAMOND', gradientStops: [{position, color}], gradientTransform }`

**ImagePaint**: `{ type: 'IMAGE', imageHash, scaleMode: 'FILL' | 'FIT' | 'CROP' | 'TILE', filters? }`

**VideoPaint**: `{ type: 'VIDEO', videoHash, scaleMode }`

### Effect Types

**DropShadow**: `{ type: 'DROP_SHADOW', color: RGBA, offset: Vector, radius, spread?, visible, blendMode }`

**InnerShadow**: `{ type: 'INNER_SHADOW', ... }` (same structure)

**Blur**: `{ type: 'LAYER_BLUR' | 'BACKGROUND_BLUR', radius }`

### BlendModes
NORMAL, MULTIPLY, SCREEN, OVERLAY, DARKEN, LIGHTEN, COLOR_DODGE, COLOR_BURN, HARD_LIGHT, SOFT_LIGHT, DIFFERENCE, EXCLUSION, HUE, SATURATION, COLOR, LUMINOSITY

---

## 6. Text Manipulation

### Font Loading (REQUIRED before any text edit)
```
await figma.loadFontAsync({ family: "Inter", style: "Regular" })
```

### Content
- `node.characters` — get/set text content
- `node.insertCharacters(start, text)` — insert
- `node.deleteCharacters(start, end)` — remove range

### Styling (whole node)
- `fontName: { family, style }` — e.g., `{ family: "Inter", style: "Bold" }`
- `fontSize` — minimum 1
- `lineHeight` — `{ value, unit: 'PIXELS' | 'PERCENT' | 'AUTO' }`
- `letterSpacing` — `{ value, unit: 'PIXELS' | 'PERCENT' }`
- `textCase` — ORIGINAL, UPPER, LOWER, TITLE, SMALL_CAPS, SMALL_CAPS_FORCED
- `textDecoration` — NONE, UNDERLINE, STRIKETHROUGH
- `textAlignHorizontal` — LEFT, CENTER, RIGHT, JUSTIFIED
- `textAlignVertical` — TOP, CENTER, BOTTOM
- `textAutoResize` — NONE, WIDTH_AND_HEIGHT, HEIGHT, TRUNCATE

### Range-based styling (per character)
Every text property has `getRangeX(start, end)` / `setRangeX(start, end, value)`:
- FontName, FontSize, TextCase, TextDecoration
- LetterSpacing, LineHeight, Fills, Hyperlink
- Allows mixed styling within a single text node

---

## 7. Auto Layout

Auto Layout on FrameNode maps to CSS Flexbox:

### Direction & Wrapping
- `layoutMode` — 'HORIZONTAL' | 'VERTICAL' | 'NONE'
- `layoutWrap` — 'NO_WRAP' | 'WRAP'

### Alignment
- `primaryAxisAlignItems` — 'MIN' | 'MAX' | 'CENTER' | 'SPACE_BETWEEN'
- `counterAxisAlignItems` — 'MIN' | 'MAX' | 'CENTER' | 'BASELINE'
- `counterAxisAlignContent` — 'AUTO' | 'SPACE_BETWEEN' (for wrap)

### Spacing
- `itemSpacing` — gap between children
- `counterAxisSpacing` — gap between wrapped tracks
- `paddingTop`, `paddingRight`, `paddingBottom`, `paddingLeft`

### Sizing Mode
- `primaryAxisSizingMode` — 'FIXED' | 'AUTO'
- `counterAxisSizingMode` — 'FIXED' | 'AUTO'

### Child Properties
- `layoutAlign` — child alignment override
- `layoutGrow` — 0 (fixed) or 1 (fill container)
- `layoutPositioning` — 'AUTO' (in flow) | 'ABSOLUTE' (positioned)
- `layoutSizingHorizontal` — 'FIXED' | 'HUG' | 'FILL'
- `layoutSizingVertical` — 'FIXED' | 'HUG' | 'FILL'

### CSS Grid (newer)
- `layoutMode: 'GRID'`
- `gridColumnCount`, `gridRowCount`
- `gridColumnGap`, `gridRowGap`
- `gridColumnSpan`, `gridRowSpan` (on children)

### Mapping to CSS

| Figma Auto Layout | CSS Flexbox |
|---|---|
| `layoutMode: 'HORIZONTAL'` | `display: flex; flex-direction: row` |
| `layoutMode: 'VERTICAL'` | `display: flex; flex-direction: column` |
| `primaryAxisAlignItems: 'MIN'` | `justify-content: flex-start` |
| `primaryAxisAlignItems: 'CENTER'` | `justify-content: center` |
| `primaryAxisAlignItems: 'MAX'` | `justify-content: flex-end` |
| `primaryAxisAlignItems: 'SPACE_BETWEEN'` | `justify-content: space-between` |
| `counterAxisAlignItems: 'MIN'` | `align-items: flex-start` |
| `counterAxisAlignItems: 'CENTER'` | `align-items: center` |
| `itemSpacing: 16` | `gap: 16px` |
| `layoutSizingHorizontal: 'FILL'` | `flex: 1` or `width: 100%` |
| `layoutSizingHorizontal: 'HUG'` | `width: fit-content` |

---

## 8. Variables and Design Tokens

### Variable Types
- BOOLEAN, COLOR, FLOAT, STRING

### CRUD
```
figma.variables.createVariableCollection(name)
figma.variables.createVariable(name, collection, type)
variable.setValueForMode(modeId, value)
variable.remove()
collection.addMode(name)
collection.removeMode(modeId)
```

### Binding to Nodes
```
node.setBoundVariable('fills/0/color', variable)
node.boundVariables  // read all bindings
node.setExplicitVariableModeForCollection(collection, modeId)
```

### Collections & Modes
- Collections group related variables (e.g., "Colors", "Spacing")
- Modes enable variants (e.g., "Light" / "Dark")
- Variables store different values per mode

---

## 9. Components and Instances

### Creating Components
```
const component = figma.createComponent()
component.name = "Button"
// Add children, set properties, styles...

// Create variant set
const set = figma.combineAsVariants([primary, secondary], parent)
```

### Component Properties
```
component.addComponentProperty("label", "TEXT", "Click me")
component.addComponentProperty("variant", "VARIANT", "primary")
component.addComponentProperty("disabled", "BOOLEAN", false)
component.addComponentProperty("icon", "INSTANCE_SWAP", defaultIconId)
```

### Instantiating
```
const instance = component.createInstance()
instance.setProperties({ "label": "Submit", "variant": "secondary" })
instance.swapComponent(otherComponent)  // swap to different component
```

### Library Import
```
const component = await figma.importComponentByKeyAsync(key)
const instance = component.createInstance()
```

---

## 10. Named Styles

### Types
- **PaintStyle** — reusable fill/stroke color configurations
- **TextStyle** — reusable text formatting (font, size, line-height, etc.)
- **EffectStyle** — reusable shadow/blur configurations
- **GridStyle** — reusable layout grid configurations

### CRUD
```
const style = figma.createPaintStyle()
style.name = "Primary/500"
style.paints = [{ type: 'SOLID', color: { r: 0.2, g: 0.4, b: 1 } }]

// Apply to node
await node.setFillStyleIdAsync(style.id)

// Read all styles
const styles = await figma.getLocalPaintStylesAsync()
```

---

## 11. Vector Editing

### VectorNetwork — Figma's path model (superset of SVG paths)

Three components:
- **Vertices** — points with x/y, optional strokeCap, cornerRadius
- **Segments** — connections between vertices, optional cubic spline tangents
- **Regions** — fillable areas defined by loops of segments

```
node.vectorNetwork = {
  vertices: [{ x: 0, y: 0 }, { x: 100, y: 0 }, { x: 50, y: 100 }],
  segments: [
    { start: 0, end: 1 },
    { start: 1, end: 2 },
    { start: 2, end: 0 }
  ],
  regions: [{ windingRule: "NONZERO", loops: [[0, 1, 2]] }]
}
```

### SVG Import
```
const frame = figma.createNodeFromSvg('<svg>...</svg>')
```

---

## 12. Prototyping

### Reactions
```
await node.setReactionsAsync([{
  trigger: { type: 'ON_CLICK' },
  actions: [{
    type: 'NODE',
    destinationId: targetFrameId,
    navigation: 'NAVIGATE',
    transition: { type: 'DISSOLVE', duration: 0.3, easing: { type: 'EASE_OUT' } }
  }]
}])
```

### Trigger Types
ON_CLICK, ON_HOVER, ON_PRESS, ON_DRAG, AFTER_TIMEOUT, MOUSE_ENTER, MOUSE_LEAVE, ON_KEY_DOWN

### Action Types
BACK, CLOSE, URL, NODE (navigate/overlay/swap/scroll_to/change_to), SET_VARIABLE, SET_VARIABLE_MODE, CONDITIONAL

---

## 13. Export

```
// PNG at 2x
const bytes = await node.exportAsync({ format: 'PNG', constraint: { type: 'SCALE', value: 2 } })

// SVG string
const svg = await node.exportAsync({ format: 'SVG_STRING' })

// CSS
const css = await node.getCSSAsync()

// PDF
const pdf = await node.exportAsync({ format: 'PDF' })
```

Formats: PNG, JPG, SVG, SVG_STRING, PDF, JSON_REST_V1

---

## 14. Events

```
figma.on('selectionchange', () => { ... })       // user changes selection
figma.on('currentpagechange', () => { ... })      // user switches pages
figma.on('documentchange', (event) => { ... })    // any document modification
figma.on('close', () => { ... })                  // plugin closing
figma.on('drop', (event) => { ... })              // item dropped on canvas
```

---

## 15. Limitations

- **One plugin at a time** per user
- **No background execution** — user must initiate
- **Dev Mode is read-only** — cannot create/modify nodes
- **Font loading required** — must `loadFontAsync()` before any text editing
- **Null origin CORS** — plugin iframes can only call APIs with `Access-Control-Allow-Origin: *`
- **Cannot access** file metadata, permissions, comments, or version history
- **Cannot programmatically** enable team libraries
- **100 KB limit** per plugin data entry
- **Component creation** limited to Design mode (not FigJam, Dev Mode)

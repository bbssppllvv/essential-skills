# Figma Plugin API Reference for AI Agents

**Based on @figma/plugin-typings v1.123.0 (Jan 2026)**

## Table of Contents
1. [Architecture](#1-architecture)
2. [Node Creation](#2-node-creation)
3. [Node Types (39)](#3-node-types)
4. [Universal Node Properties](#4-universal-node-properties)
5. [Fills, Strokes, Effects](#5-fills-strokes-effects)
6. [Text Manipulation](#6-text-manipulation)
7. [Auto Layout & CSS Grid](#7-auto-layout--css-grid)
8. [Variables and Design Tokens](#8-variables-and-design-tokens)
9. [Components and Instances](#9-components-and-instances)
10. [Named Styles](#10-named-styles)
11. [Vector Editing](#11-vector-editing)
12. [Prototyping](#12-prototyping)
13. [Export](#13-export)
14. [Events](#14-events)
15. [Limitations](#15-limitations)

**See also:**
- `references/new-effects-paints.md` — Deep dive on Glass, Noise, Texture, Progressive Blur, Pattern Paint (2025+ beta effects with code examples and gotchas)
- `references/advanced-apis.md` — CSS Grid, Annotations, Dev Resources, Measurements, Codegen, Payments, Team Library, Slides, Buzz, Draw, JSX, script execution, data storage, REST API, webhooks, deprecations

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
figma.createText()                                     → TextNode
figma.createTextPath(node, startSegment, startPos)     → TextPathNode (NEW — text on a path)
```

### Containers
```
figma.createFrame()              → FrameNode
figma.createComponent()          → ComponentNode (Design only)
figma.createComponentFromNode()  → ComponentNode (Design only)
figma.createPage()               → PageNode (Design only)
figma.createPageDivider(name?)   → PageNode (NEW — page divider)
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
figma.transformGroup(nodes, parent, index, modifiers) → TransformGroupNode (NEW)
```

### Media & Content
```
figma.createImage(data: Uint8Array)      → Image
figma.createImageAsync(src: string)      → Promise<Image>
figma.createVideoAsync(data: Uint8Array) → Promise<Video>
figma.createNodeFromSvg(svg: string)     → FrameNode
figma.createNodeFromJSXAsync(jsx)        → Promise<SceneNode>
figma.loadBrushesAsync(type)             → Promise<void> (NEW — 'STRETCH' | 'SCATTER')
```

### Slides
```
figma.createSlide(row?, col?)    → SlideNode (NEW)
figma.createSlideRow(row?)       → SlideRowNode (NEW)
```

### FigJam Only
```
figma.createSticky()             → StickyNode
figma.createShapeWithText()      → ShapeWithTextNode
figma.createConnector()          → ConnectorNode
figma.createCodeBlock()          → CodeBlockNode
figma.createTable(rows, cols)    → TableNode
figma.createLinkPreviewAsync(url) → Promise<EmbedNode | LinkUnfurlNode>
figma.createGif(hash)            → MediaNode
```

### Styles
```
figma.createPaintStyle()         → PaintStyle
figma.createTextStyle()          → TextStyle
figma.createEffectStyle()        → EffectStyle
figma.createGridStyle()          → GridStyle
```

---

## 3. Node Types (33 SceneNode types)

`BaseNode = DocumentNode | PageNode | SceneNode`

| Type | Context | Description |
|---|---|---|
| DocumentNode | All | Root of document (NOT a SceneNode) |
| PageNode | All | Individual pages (NOT a SceneNode) |

SceneNode union (33 types):

| Type | Context | Description |
|---|---|---|
| FrameNode | All | Primary container, supports auto layout + grid |
| GroupNode | All | Simple grouping |
| SectionNode | All | Organization container |
| RectangleNode | All | Rectangle |
| EllipseNode | All | Ellipse/circle |
| PolygonNode | All | Regular polygon |
| StarNode | All | Star shape |
| LineNode | All | Line |
| VectorNode | All | Custom vector paths |
| TextNode | All | Text content |
| TextPathNode | All | Text on a path (NEW 2025) |
| TransformGroupNode | All | Group with transform modifiers (NEW 2025) |
| ComponentNode | Design | Reusable component definition |
| ComponentSetNode | Design | Variant set container |
| InstanceNode | All | Component instance |
| BooleanOperationNode | All | Boolean op result |
| SliceNode | All | Export region |
| StickyNode | FigJam | Sticky note |
| ConnectorNode | FigJam | Connector line |
| ShapeWithTextNode | FigJam | Shape with text |
| CodeBlockNode | FigJam | Code block |
| TableNode | FigJam | Table (cells accessed via TableNode.cellAt()) |
| StampNode | FigJam | Stamp |
| HighlightNode | FigJam | Highlight |
| WashiTapeNode | FigJam | Washi tape |
| WidgetNode | All | Widget instance |
| EmbedNode | FigJam | Embedded content |
| LinkUnfurlNode | FigJam | Link preview |
| MediaNode | FigJam | Media element |
| SlideNode | Slides | Presentation slide (NEW 2025) |
| SlideRowNode | Slides | Slide row (NEW 2025) |
| SlideGridNode | Slides | Slide grid (NEW 2025) |
| InteractiveSlideElementNode | Slides | Interactive element (NEW 2025) |

Note: TableCellNode exists but is NOT a SceneNode. Access cells via `TableNode.cellAt(row, col)`.

Editor types: `figma.editorType` → `'figma' | 'figjam' | 'dev' | 'slides' | 'buzz'`

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

### Paint Types (5)

`type Paint = SolidPaint | GradientPaint | ImagePaint | VideoPaint | PatternPaint`

**SolidPaint**: `{ type: 'SOLID', color: {r, g, b}, opacity?, visible?, blendMode? }`

**GradientPaint**: `{ type: 'GRADIENT_LINEAR' | 'GRADIENT_RADIAL' | 'GRADIENT_ANGULAR' | 'GRADIENT_DIAMOND', gradientStops: [{position, color}], gradientTransform }`

**ImagePaint**: `{ type: 'IMAGE', imageHash, scaleMode: 'FILL' | 'FIT' | 'CROP' | 'TILE', filters? }`

**VideoPaint**: `{ type: 'VIDEO', videoHash, scaleMode }`

**PatternPaint** (NEW beta): `{ type: 'PATTERN', sourceNodeId, tileType: 'RECTANGULAR' | 'HORIZONTAL_HEXAGONAL' | 'VERTICAL_HEXAGONAL', scalingFactor, spacing: Vector, horizontalAlignment }` — Must use `await node.setFillsAsync()`

### Effect Types (6)

`type Effect = DropShadowEffect | InnerShadowEffect | BlurEffect | NoiseEffect | TextureEffect | GlassEffect`

**DropShadow**: `{ type: 'DROP_SHADOW', color: RGBA, offset: Vector, radius, spread?, visible, blendMode }` (max 8/layer)

**InnerShadow**: `{ type: 'INNER_SHADOW', ... }` (same structure, max 8/layer)

**Blur**: `{ type: 'LAYER_BLUR' | 'BACKGROUND_BLUR', blurType: 'NORMAL' | 'PROGRESSIVE', radius }` (max 1/layer)
- Progressive: adds `startRadius`, `startOffset: Vector`, `endOffset: Vector` (normalized 0-1 object space)

**NoiseEffect** (NEW beta): `{ type: 'NOISE', noiseType: 'MONOTONE' | 'DUOTONE' | 'MULTITONE', color, noiseSize, density, blendMode }` (max 2/layer)

**TextureEffect** (NEW beta): `{ type: 'TEXTURE', noiseSize, radius, clipToShape }` (max 1/layer)

**GlassEffect** (NEW beta): `{ type: 'GLASS', lightIntensity, lightAngle, refraction, depth, dispersion, radius }` (max 1/layer, cannot combine with BACKGROUND_BLUR)

See `references/new-effects-paints.md` for full type definitions, code examples, and gotchas.

### StrokeCap
`'NONE' | 'ROUND' | 'SQUARE' | 'ARROW_LINES' | 'ARROW_EQUILATERAL' | 'DIAMOND_FILLED' | 'TRIANGLE_FILLED' | 'CIRCLE_FILLED'`

### BlendModes (20)
PASS_THROUGH, NORMAL, DARKEN, MULTIPLY, LINEAR_BURN, COLOR_BURN, LIGHTEN, SCREEN, LINEAR_DODGE, COLOR_DODGE, OVERLAY, SOFT_LIGHT, HARD_LIGHT, DIFFERENCE, EXCLUSION, HUE, SATURATION, COLOR, LUMINOSITY

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
- `textAutoResize` — NONE, WIDTH_AND_HEIGHT, HEIGHT (TRUNCATE is **deprecated** — use `textTruncation` instead)
- `textTruncation` — DISABLED, ENDING (controls ellipsis truncation). With `textAutoResize: 'NONE'`, truncates when fixed size < text. With `'HEIGHT'` or `'WIDTH_AND_HEIGHT'`, truncation only occurs in conjunction with `maxHeight` or `maxLines`.
- `maxLines` — `number | null` (max lines before truncation; only applies when `textTruncation` is `'ENDING'`; value must be >= 1, set to `null` to disable)

### Range-based styling (per character)
Every text property has `getRangeX(start, end)` / `setRangeX(start, end, value)`:
- FontName, FontSize, TextCase, TextDecoration
- LetterSpacing, LineHeight, Fills, Hyperlink
- Allows mixed styling within a single text node

---

## 7. Auto Layout & CSS Grid

Auto Layout on FrameNode maps to CSS Flexbox. Grid Layout maps to CSS Grid.

### Three API Layers

Figma has three layers of sizing control. Understanding which to use is critical:

1. **Container properties** (`AutoLayoutMixin`) — on the auto-layout frame itself: `primaryAxisSizingMode`, `counterAxisSizingMode`
2. **Child properties** (`AutoLayoutChildrenMixin`) — on direct children: `layoutGrow`, `layoutAlign`, `layoutPositioning`
3. **Shorthand properties** (`LayoutMixin`) — orientation-independent: `layoutSizingHorizontal`, `layoutSizingVertical`

The shorthands are the **preferred modern API**. They internally set `layoutGrow`, `layoutAlign`, `primaryAxisSizingMode`, and `counterAxisSizingMode`.

### Axis Terminology

| layoutMode | Primary Axis | Counter Axis |
|---|---|---|
| `HORIZONTAL` | Horizontal (X) — grows as items added | Vertical (Y) |
| `VERTICAL` | Vertical (Y) — grows as items added | Horizontal (X) |

### Direction & Wrapping
- `layoutMode` — `'HORIZONTAL' | 'VERTICAL' | 'GRID' | 'NONE'` (default `'NONE'`). Setting this repositions children as a side effect. Toggling back to `'NONE'` does NOT restore original positions. **Must be set FIRST** before other auto-layout properties.
- `layoutWrap` — `'NO_WRAP' | 'WRAP'`. **Only valid on `layoutMode: 'HORIZONTAL'`** — throws on VERTICAL or GRID.

### Alignment
- `primaryAxisAlignItems` — `'MIN' | 'MAX' | 'CENTER' | 'SPACE_BETWEEN'` (only HORIZONTAL/VERTICAL, NOT GRID)
- `counterAxisAlignItems` — `'MIN' | 'MAX' | 'CENTER' | 'BASELINE'` (only HORIZONTAL/VERTICAL). `'BASELINE'` only valid on HORIZONTAL frames.
- `counterAxisAlignContent` — `'AUTO' | 'SPACE_BETWEEN'` — **only on WRAP frames**. Throws if set on non-wrap.

### Spacing
- `itemSpacing` — gap between children (HORIZONTAL/VERTICAL only, NOT GRID — use `gridRowGap`/`gridColumnGap` for GRID)
- `counterAxisSpacing` — `number | null` — gap between wrapped tracks (only with `layoutWrap: 'WRAP'`). Setting to `null` syncs with `itemSpacing`.
- `paddingTop`, `paddingRight`, `paddingBottom`, `paddingLeft` — work on ALL auto-layout frames including GRID. (`horizontalPadding`/`verticalPadding` are **deprecated**.)

### Container Sizing Mode
- `primaryAxisSizingMode` — `'FIXED' | 'AUTO'`. `'AUTO'` = hug children along primary axis. `'AUTO'` **must NOT** be used when any child has `layoutGrow = 1` or `layoutAlign = 'STRETCH'`.
- `counterAxisSizingMode` — `'FIXED' | 'AUTO'`. Same constraint as primary.

### Shorthand Sizing (preferred API)

`layoutSizingHorizontal` / `layoutSizingVertical` — `'FIXED' | 'HUG' | 'FILL'`

**Validity constraints:**
- `'HUG'` — only valid on **auto-layout frames** and **text nodes**. Throws on other nodes.
- `'FILL'` — only valid on **direct children of auto-layout frames**. Throws on top-level frames or nodes outside auto-layout.
- `'FIXED'` — always valid.

**How shorthands map to old properties:**

On a container (auto-layout frame itself):
| Shorthand | HORIZONTAL layout sets | VERTICAL layout sets |
|---|---|---|
| `layoutSizingHorizontal = 'HUG'` | `primaryAxisSizingMode = 'AUTO'` | `counterAxisSizingMode = 'AUTO'` |
| `layoutSizingHorizontal = 'FIXED'` | `primaryAxisSizingMode = 'FIXED'` | `counterAxisSizingMode = 'FIXED'` |
| `layoutSizingVertical = 'HUG'` | `counterAxisSizingMode = 'AUTO'` | `primaryAxisSizingMode = 'AUTO'` |
| `layoutSizingVertical = 'FIXED'` | `counterAxisSizingMode = 'FIXED'` | `primaryAxisSizingMode = 'FIXED'` |

On a child of auto-layout frame:
| Shorthand | HORIZONTAL parent sets | VERTICAL parent sets |
|---|---|---|
| `layoutSizingHorizontal = 'FILL'` | `layoutGrow = 1` | `layoutAlign = 'STRETCH'` |
| `layoutSizingHorizontal = 'FIXED'` | `layoutGrow = 0` | `layoutAlign = 'INHERIT'` |
| `layoutSizingVertical = 'FILL'` | `layoutAlign = 'STRETCH'` | `layoutGrow = 1` |
| `layoutSizingVertical = 'FIXED'` | `layoutAlign = 'INHERIT'` | `layoutGrow = 0` |

Key insight: **FILL on primary axis = `layoutGrow = 1`; FILL on counter axis = `layoutAlign = 'STRETCH'`**.

### Legacy Child Properties (still functional, not deprecated)

- `layoutGrow` — `0` or `1` only. Controls primary axis of parent. `0` = fixed, `1` = fill.
- `layoutAlign` — `'MIN' | 'CENTER' | 'MAX' | 'STRETCH' | 'INHERIT'`. Controls counter axis of parent. `'STRETCH'` = fill. Values `'MIN'`/`'CENTER'`/`'MAX'` are **DEPRECATED** — use parent's `counterAxisAlignItems` instead.
- `layoutPositioning` — `'AUTO'` (in layout flow) | `'ABSOLUTE'` (removed from flow, uses x/y + constraints)

### Nested Auto-Layout Conflict

When an auto-layout frame is FILL inside another auto-layout frame, you **must** set its own sizing mode to FIXED on that axis. A frame **cannot simultaneously stretch to fill parent AND shrink to hug children**. Example: if `layoutGrow = 1`, set `primaryAxisSizingMode = 'FIXED'`.

### Min/Max Size Constraints

- `minWidth`, `maxWidth`, `minHeight`, `maxHeight` — `number | null`
- Only work on **auto-layout frames and their direct children**
- Must be **positive** (not 0 or negative). Set to `null` to remove.
- **Must be applied AFTER `appendChild()`** — they require auto-layout context

### Other Layout Properties

- `strokesIncludedInLayout` — `boolean`. When `true`, strokes included in size calculation (like CSS `box-sizing: border-box`).
- `itemReverseZIndex` — `boolean`. When `true`, first child renders on top (for overlapping with negative `itemSpacing`).
- `clipsContent` — `boolean`. Whether frame clips overflow.
- `overflowDirection` — `'NONE' | 'HORIZONTAL' | 'VERTICAL' | 'BOTH'`. Controls scroll behavior in presentation mode ("Overflow Behavior" in Prototype tab). Frames directly under the canvas scroll automatically when larger than the viewport.
- `numberOfFixedChildren` — `number`. How many children (counted from the **end** of the children array) are fixed when scrolling. Fixed children render on top of scrolling content. Set this instead of a per-child boolean.
- `inferredAutoLayout` (readonly) — `InferredAutoLayoutResult | null`. Returns heuristically inferred auto-layout properties for frames without explicit auto-layout (used by Dev Mode code snippets). Returns `null` if not applicable. Useful for reading/analyzing manually laid-out frames.

### Common Sizing Patterns

| Use case | Width | Height |
|---|---|---|
| **Artboard (scrollable page)** | `FIXED` (1440px) | `HUG` (grows with content) |
| **Artboard (fixed viewport)** | `FIXED` (1440px) | `FIXED` (900px) |
| **Section/row inside page** | `FILL` (stretch to artboard) | `HUG` (fit content) |
| **Card in a grid** | `FILL` (share space) | `HUG` (fit content) |
| **Button** | `HUG` (fit label) | `HUG` (fit label) |
| **Full-width button** | `FILL` (stretch) | `HUG` (fit label) |
| **Sidebar** | `FIXED` (280px) | `FILL` (match parent) |
| **Text paragraph** | `FILL` (wrap in parent) | `HUG` (grow with text) |

### Text Nodes in Auto Layout

Text nodes have BOTH `textAutoResize` and `layoutSizingHorizontal`/`layoutSizingVertical`. These are **independent properties** — neither automatically sets the other — but they must be coordinated for correct behavior.

`textAutoResize` controls the text box's intrinsic sizing. `layoutSizing*` controls how the node participates in its parent's auto-layout. When both apply, keep them consistent:

| Desired behavior | `textAutoResize` | `layoutSizingHorizontal` | `layoutSizingVertical` |
|---|---|---|---|
| Text grows to fit content (no wrap) | `WIDTH_AND_HEIGHT` | `HUG` | `HUG` |
| Text wraps at fixed/fill width | `HEIGHT` | `FIXED` or `FILL` | `HUG` |
| Fixed text box (no auto-sizing) | `NONE` | `FIXED` | `FIXED` |

**Key gotchas:**
- If `layoutSizingHorizontal = 'FILL'` but `textAutoResize = 'WIDTH_AND_HEIGHT'`, text will not wrap — it expands infinitely. Set `textAutoResize = 'HEIGHT'` when text fills horizontally.
- Figma may reset `textAutoResize` when appending text nodes to Grid containers. Re-apply `textAutoResize` after `appendChild()` in Grid contexts.
- `maxWidth` on a text node is ignored when `textAutoResize = 'WIDTH_AND_HEIGHT'` — switch to `HEIGHT` for `maxWidth` to take effect.

### CSS Grid (Update 115, July 2025)

Container: `layoutMode: 'GRID'`
- `gridColumnCount`, `gridRowCount`, `gridColumnGap`, `gridRowGap`
- `gridColumnSizes`, `gridRowSizes` — `Array<GridTrackSize>`. **Mutable** — you can set `.type` and `.value` directly on returned objects (unlike most Figma properties that require clone-and-reassign).
- `appendChildAt(child, row, col)` — place child at specific cell. Regular `appendChild()` auto-places to first available cell.

Child:
- `gridColumnSpan`, `gridRowSpan` — number of cells spanned
- `gridColumnAnchorIndex`, `gridRowAnchorIndex` — **readonly**, use `setGridChildPosition()` to change
- `gridChildHorizontalAlign`, `gridChildVerticalAlign` — `'MIN' | 'CENTER' | 'MAX' | 'AUTO'`
- `setGridChildPosition(row, col)`

Grid-specific constraints:
- `primaryAxisAlignItems`/`counterAxisAlignItems` do NOT apply to GRID — use per-child `gridChildHorizontalAlign`/`gridChildVerticalAlign`
- `itemSpacing` does NOT apply to GRID — use `gridRowGap`/`gridColumnGap`
- FLEX tracks cannot be used when container sizing is HUG

See `references/advanced-apis.md` section 1 for full details and code examples.

### Plugin Ordering Requirements

1. Set `layoutMode` **first** before any other auto-layout properties
2. Set `layoutWrap` only on HORIZONTAL frames
3. Set `counterAxisSpacing`/`counterAxisAlignContent` only after `layoutWrap = 'WRAP'`
4. Apply `minWidth`/`maxWidth`/`minHeight`/`maxHeight` **after `appendChild()`**
5. Do not use `primaryAxisSizingMode = 'AUTO'` with `layoutGrow = 1` or `layoutAlign = 'STRETCH'` on the same axis

### Mapping to CSS

| Figma Auto Layout | CSS Flexbox |
|---|---|
| `layoutMode: 'HORIZONTAL'` | `display: flex; flex-direction: row` |
| `layoutMode: 'VERTICAL'` | `display: flex; flex-direction: column` |
| `layoutWrap: 'WRAP'` | `flex-wrap: wrap` |
| `primaryAxisAlignItems: 'MIN'` | `justify-content: flex-start` |
| `primaryAxisAlignItems: 'CENTER'` | `justify-content: center` |
| `primaryAxisAlignItems: 'MAX'` | `justify-content: flex-end` |
| `primaryAxisAlignItems: 'SPACE_BETWEEN'` | `justify-content: space-between` |
| `counterAxisAlignItems: 'MIN'` | `align-items: flex-start` |
| `counterAxisAlignItems: 'CENTER'` | `align-items: center` |
| `counterAxisAlignItems: 'BASELINE'` | `align-items: baseline` |
| `counterAxisAlignContent: 'SPACE_BETWEEN'` | `align-content: space-between` |
| `itemSpacing: 16` | `gap: 16px` |
| `layoutSizingHorizontal: 'FILL'` | `flex: 1` or `width: 100%` |
| `layoutSizingHorizontal: 'HUG'` | `width: fit-content` |
| `layoutSizingHorizontal: 'FIXED'` | `width: Npx` |
| `layoutSizingVertical: 'FILL'` | `flex: 1` (cross axis) or `height: 100%` |
| `layoutSizingVertical: 'HUG'` | `height: auto` / `height: fit-content` |
| `primaryAxisSizingMode: 'AUTO'` | primary axis `fit-content` (hug) |
| `counterAxisSizingMode: 'AUTO'` | counter axis `fit-content` (hug) |
| `layoutPositioning: 'ABSOLUTE'` | `position: absolute` |
| `strokesIncludedInLayout: true` | `box-sizing: border-box` |
| `minWidth: 200` | `min-width: 200px` |
| `maxWidth: 600` | `max-width: 600px` |

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
node.setBoundVariable(field, variable)   // bind node-level or text field
node.boundVariables                       // read all bindings
node.setExplicitVariableModeForCollection(collection, modeId)

// Paint/effect/grid bindings (returns a new copy — reassign to node)
figma.variables.setBoundVariableForPaint(paint, field, variable)     → SolidPaint
figma.variables.setBoundVariableForEffect(effect, field, variable)   → Effect
figma.variables.setBoundVariableForLayoutGrid(grid, field, variable) → LayoutGrid
```

### Variable-Bindable Properties (complete list from typings)

**Node fields** (`VariableBindableNodeField` — used with `node.setBoundVariable()`):
- Dimensions: `width`, `height`, `minWidth`, `maxWidth`, `minHeight`, `maxHeight`
- Layout spacing: `itemSpacing`, `counterAxisSpacing`, `paddingTop`, `paddingRight`, `paddingBottom`, `paddingLeft`
- Grid spacing: `gridRowGap`, `gridColumnGap`
- Corner radius: `topLeftRadius`, `topRightRadius`, `bottomLeftRadius`, `bottomRightRadius`
- Stroke: `strokeWeight`, `strokeTopWeight`, `strokeRightWeight`, `strokeBottomWeight`, `strokeLeftWeight`
- Visual: `opacity`, `visible`
- Content: `characters`

**Text fields** (`VariableBindableTextField` — used with `node.setBoundVariable()` or `node.setRangeBoundVariable()`):
- `fontFamily`, `fontSize`, `fontStyle`, `fontWeight`
- `letterSpacing`, `lineHeight`, `paragraphSpacing`, `paragraphIndent`

**Paint fields** (`VariableBindablePaintField` — used with `setBoundVariableForPaint()`):
- `color`

**Effect fields** (`VariableBindableEffectField` — used with `setBoundVariableForEffect()`):
- `color`, `radius`, `spread`, `offsetX`, `offsetY`

**Layout grid fields** (`VariableBindableLayoutGridField` — used with `setBoundVariableForLayoutGrid()`):
- `sectionSize`, `count`, `offset`, `gutterSize`

**Gradient stop fields** (`VariableBindableColorStopField`): `color`

**Component property fields** (`VariableBindableComponentPropertyField`): `value`

**Style-level binding** (whole-style variable binding on named styles):
- PaintStyle: `paints` | EffectStyle: `effects` | GridStyle: `layoutGrids`

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
ON_CLICK, ON_HOVER, ON_PRESS, ON_DRAG, AFTER_TIMEOUT, MOUSE_UP, MOUSE_DOWN, MOUSE_ENTER, MOUSE_LEAVE, ON_KEY_DOWN, ON_MEDIA_HIT (NEW), ON_MEDIA_END (NEW)

### Action Types
BACK, CLOSE, URL, NODE (navigate/overlay/swap/scroll_to/change_to), UPDATE_MEDIA_RUNTIME (NEW), SET_VARIABLE, SET_VARIABLE_MODE, CONDITIONAL

### Easing Types
EASE_IN, EASE_OUT, EASE_IN_AND_OUT, LINEAR, EASE_IN_BACK, EASE_OUT_BACK, EASE_IN_AND_OUT_BACK, CUSTOM_CUBIC_BEZIER, GENTLE (NEW spring), QUICK (NEW spring), BOUNCY (NEW spring), SLOW (NEW spring), CUSTOM_SPRING (NEW)

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

### figma.on() events
```
'selectionchange'     // user changes selection
'currentpagechange'   // user switches pages
'documentchange'      // any document modification (DocumentChangeEvent)
'close'               // plugin closing
'drop'                // item dropped on canvas (DropEvent → boolean)
'run'                 // plugin started (RunEvent)
'slidesviewchange'    // slides view changed (NEW)
'canvasviewchange'    // canvas view changed (NEW)
'textreview'          // text review triggered (NEW)
'stylechange'         // style changed (NEW)
'timerstart/stop/pause/resume/adjust/done'  // FigJam timer events
```

### Other event sources
```
figma.codegen.on('generate', ...)         // Dev Mode codegen
figma.devResources.on('linkpreview', ...) // Dev Mode link preview
figma.parameters.on('input', ...)         // parameter input UI
page.on('nodechange', ...)                // per-page node changes
figma.ui.on('message', ...)               // UI thread messages
```

---

## 15. Limitations

- **One plugin at a time** per user
- **No background execution** — user must initiate
- **Dev Mode is read-only** — cannot create/modify design nodes
- **Font loading required** — must `loadFontAsync()` before any text editing
- **Null origin CORS** — plugin iframes can only call APIs with `Access-Control-Allow-Origin: *`
- **No fetch in main thread** — network calls must go through UI iframe
- **Cannot programmatically** enable team libraries (must enable via UI)
- **100 KB limit** per plugin data entry, 5 MB total client storage
- **Component creation** limited to Design mode (not FigJam, Dev Mode)
- **Dynamic page loading** — new plugins MUST use async APIs (since April 2024)
- **Codegen callback timeout** — 15 seconds max for codegen plugin responses
- **Glass effect** — Frame-only in API, variable binding not supported
- **PatternPaint** — must use `setFillsAsync()`, direct assignment throws
- **New effects (Noise, Texture, Glass, Progressive Blur)** — all beta, API may change

See `references/advanced-apis.md` section 19 for full deprecation list and sync→async migration guide.

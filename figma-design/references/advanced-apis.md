# Figma Plugin API — Advanced APIs Reference (2024-2025)

APIs beyond core design manipulation: annotations, measurements, dev resources, codegen, payments, team library, slides, buzz, draw, script execution, data storage, and JSX node creation.

---

## Table of Contents
1. [CSS Grid Layout](#1-css-grid-layout)
2. [Annotations](#2-annotations)
3. [Dev Resources](#3-dev-resources)
4. [Measurements](#4-measurements)
5. [Codegen Plugins](#5-codegen-plugins)
6. [Payments API](#6-payments-api)
7. [Team Library](#7-team-library)
8. [Extended Variable Collections](#8-extended-variable-collections)
9. [Slides API](#9-slides-api)
10. [Buzz API](#10-buzz-api)
11. [Draw APIs](#11-draw-apis)
12. [JSX Node Creation](#12-jsx-node-creation)
13. [Script Execution & Runtime](#13-script-execution--runtime)
14. [Data Storage](#14-data-storage)
15. [Dynamic Page Loading](#15-dynamic-page-loading)
16. [New Text Features](#16-new-text-features)
17. [New Prototyping Features](#17-new-prototyping-features)
18. [New Node Types Summary](#18-new-node-types-summary)
19. [Deprecations & Breaking Changes](#19-deprecations--breaking-changes)
20. [REST API Additions](#20-rest-api-additions)
21. [Webhooks](#21-webhooks)

---

## 1. CSS Grid Layout — Update 115 (July 2025), HUG in Update 120 (Nov 2025)

Set `layoutMode = 'GRID'` on a frame. Values: `'NONE' | 'HORIZONTAL' | 'VERTICAL' | 'GRID'`.

### Container Properties (FrameNode, ComponentNode, etc.)

```typescript
gridRowCount: number              // min 1
gridColumnCount: number           // min 1
gridRowGap: number                // gap between rows (variable-bindable)
gridColumnGap: number             // gap between columns (variable-bindable)
gridRowSizes: GridTrackSize[]     // per-row sizing
gridColumnSizes: GridTrackSize[]  // per-column sizing
appendChildAt(child: SceneNode, rowIndex: number, columnIndex: number): void
```

### Child Properties

```typescript
gridRowSpan: number                                           // rows spanned
gridColumnSpan: number                                        // columns spanned
gridRowAnchorIndex: number                                    // row position (0-based, readonly)
gridColumnAnchorIndex: number                                 // column position (0-based, readonly)
gridChildHorizontalAlign: 'MIN' | 'CENTER' | 'MAX' | 'AUTO'
gridChildVerticalAlign: 'MIN' | 'CENTER' | 'MAX' | 'AUTO'
setGridChildPosition(rowIndex: number, columnIndex: number): void
```

### GridTrackSize

```typescript
interface GridTrackSize {
  type: 'FLEX' | 'FIXED' | 'HUG'  // HUG added Update 120
  value?: number                    // px for FIXED, fr for FLEX
}
```

### Code Example

```typescript
const grid = figma.createFrame()
grid.resize(800, 600)
grid.layoutMode = 'GRID'
grid.gridColumnCount = 3
grid.gridRowCount = 2
grid.gridColumnGap = 16
grid.gridRowGap = 16
grid.gridColumnSizes = [
  { type: 'FLEX', value: 1 },
  { type: 'FLEX', value: 2 },  // 2fr — double width
  { type: 'FIXED', value: 200 }
]
grid.gridRowSizes = [
  { type: 'HUG' },             // fit-content(100%)
  { type: 'FLEX', value: 1 }
]

const child = figma.createFrame()
grid.appendChildAt(child, 0, 0)  // place at row 0, col 0
child.gridColumnSpan = 2          // span 2 columns
```

### Gotchas
- Cannot reduce row/column count if occupied cells have children
- Cannot use FLEX tracks when container sizing is HUG
- Plugin API defaults new grids to FIXED container with FLEX tracks (UI defaults to HUG)
- Grid gap is variable-bindable (Update 117)
- `gridRowSizes`/`gridColumnSizes` return **mutable** `GridTrackSize` objects — you can set `.type`/`.value` directly (unlike fills/effects which require clone-and-reassign)
- Regular `appendChild()` on a GRID frame auto-places the child to the first available cell
- `primaryAxisAlignItems`/`counterAxisAlignItems`/`itemSpacing` do **NOT** apply to GRID frames — use `gridChildHorizontalAlign`/`gridChildVerticalAlign` for per-cell alignment, and `gridRowGap`/`gridColumnGap` for gaps
- Both `appendChildAt()` and `setGridChildPosition()` throw on out-of-bounds indices and overlap with existing children

---

## 2. Annotations — Update 85 (Jan 2024), expanded Update 104 (Dec 2024)

Notes and pinned properties on nodes for Dev Mode.

```typescript
interface Annotation {
  readonly label?: string
  readonly labelMarkdown?: string
  readonly properties?: ReadonlyArray<AnnotationProperty>
  readonly categoryId?: string
}

// 33 AnnotationPropertyType values including:
// 'width', 'height', 'fills', 'strokes', 'effects', 'cornerRadius',
// 'fontSize', 'fontFamily', 'lineHeight', 'itemSpacing', 'padding',
// 'gridRowGap', 'gridColumnGap', 'gridRowSpan', 'gridColumnSpan', ...
```

### Annotation Categories (April 2025)

```typescript
figma.annotations.getAnnotationCategoriesAsync(): Promise<AnnotationCategory[]>
figma.annotations.getAnnotationCategoryByIdAsync(id): Promise<AnnotationCategory | null>
figma.annotations.addAnnotationCategoryAsync({ label, color }): Promise<AnnotationCategory>
```

### Usage
```typescript
node.annotations  // ReadonlyArray<Annotation>
```

---

## 3. Dev Resources

Link nodes to code, docs, or external resources in Dev Mode.

```typescript
interface DevResource {
  readonly name: string
  readonly url: string
  readonly inheritedNodeId?: string  // on instances — link from main component
  nodeId: string
}

node.getDevResourcesAsync(options?: { includeChildren?: boolean }): Promise<DevResource[]>
node.addDevResourceAsync(url: string): Promise<void>
node.editDevResourceAsync(currentUrl, { name?, url? }): Promise<void>
node.deleteDevResourceAsync(url: string): Promise<void>
```

---

## 4. Measurements — Dev Mode Only

Pinned distances between nodes.

```typescript
interface Measurement {
  id: string
  start: { node: SceneNode; side: 'TOP' | 'BOTTOM' | 'LEFT' | 'RIGHT' }
  end: { node: SceneNode; side: 'TOP' | 'BOTTOM' | 'LEFT' | 'RIGHT' }
  offset: { type: 'INNER' | 'OUTER'; relative?: number; fixed?: number }
  freeText: string
}

// PageNode methods (figma.editorType === 'dev')
page.getMeasurements(): Measurement[]
page.getMeasurementsForNode(node): Measurement[]
page.addMeasurement(start, end, options?): Measurement
page.editMeasurement(id, newValue): void
page.deleteMeasurement(id): void
```

Constraint: measurements must be on same axis (LEFT↔RIGHT or TOP↔BOTTOM).

---

## 5. Codegen Plugins — Dev Mode

Custom code generation for Dev Mode inspect panel.

```typescript
figma.codegen.on('generate', (event: CodegenEvent) => CodegenResult[] | Promise<CodegenResult[]>)
figma.codegen.on('preferenceschange', (event) => Promise<void>)

interface CodegenResult {
  title: string
  code: string
  language: 'TYPESCRIPT' | 'JAVASCRIPT' | 'HTML' | 'CSS' | 'JSON' | 'PYTHON'
    | 'GO' | 'RUST' | 'SWIFT' | 'KOTLIN' | 'RUBY' | 'CPP' | 'SQL'
    | 'GRAPHQL' | 'BASH' | 'PLAINTEXT'
}
```

**Variable code syntax:**
```typescript
variable.codeSyntax: { readonly [platform in 'WEB' | 'ANDROID' | 'iOS']?: string }
variable.setVariableCodeSyntax(platform, syntax): void
variable.removeVariableCodeSyntax(platform): void
```

Callback timeout: 15 seconds (extended from 3s in April 2025).

---

## 6. Payments API

Requires `"permissions": ["payments"]` in manifest.json.

```typescript
figma.payments.status: { type: 'UNPAID' | 'PAID' | 'NOT_SUPPORTED' }
figma.payments.initiateCheckoutAsync(options?): Promise<void>
figma.payments.getUserFirstRanSecondsAgo(): number
figma.payments.setPaymentStatusInDevelopment(status): void  // dev only
figma.payments.getPluginPaymentTokenAsync(): Promise<string>
```

---

## 7. Team Library

Requires `"permissions": ["teamlibrary"]` in manifest.json.

```typescript
figma.teamLibrary.getAvailableLibraryVariableCollectionsAsync(): Promise<LibraryVariableCollection[]>
figma.teamLibrary.getVariablesInLibraryCollectionAsync(key): Promise<LibraryVariable[]>
```

Returns descriptors only (key, name, library name) — not modes or IDs. Libraries must be enabled via UI first.

---

## 8. Extended Variable Collections — Update 121 (Nov 2025), Enterprise Only

Design system theming through variable collection extension.

```typescript
figma.variables.extendLibraryCollectionByKeyAsync(key, name): Promise<ExtendedVariableCollection>
collection.extend(name): ExtendedVariableCollection
variable.valuesByModeForCollectionAsync(collection): Promise<...>
variable.removeOverrideForMode(extendedModeId): void
extendedCollection.removeOverridesForVariable(variableId): void
extendedCollection.variableOverrides  // mapping of overridden values
mode.parentModeId  // reference to parent collection mode
```

---

## 9. Slides API — Update 108 (Feb 2025)

`figma.editorType` returns `'slides'`.

```typescript
// Node types
SlideNode       // 1920x1080, non-resizable. Created: figma.createSlide(row?, col?)
SlideRowNode    // Created: figma.createSlideRow(row?)
SlideGridNode   // Exactly one per document, children are SlideRowNode[]

// Viewport
figma.viewport.slidesMode  // grid vs single-slide view

// Page
figma.currentPage.focusedSlide

// Grid manipulation
figma.getSlideGrid(): Array<Array<SlideNode>>
figma.setSlideGrid(grid): void

// Transitions
slide.getSlideTransition(): SlideTransition
slide.setSlideTransition(transition): void
slide.isSkippedSlide: boolean
```

---

## 10. Buzz API — Update 119 (Oct 2025)

Marketing asset creation tool. `figma.editorType` returns `'buzz'`.

```typescript
figma.buzz.createFrame(): FrameNode
figma.buzz.getBuzzAssetTypeForNode(node): string  // 42+ platform types
figma.buzz.setBuzzAssetTypeForNode(node, type): void
figma.buzz.getTextContent(node): ...
figma.buzz.getMediaContent(node): ...
figma.buzz.smartResize(node, width, height): void

figma.viewport.canvasView   // grid vs single-asset view
figma.currentPage.focusedNode

// Canvas grid management
figma.getCanvasGrid(): Array<Array<SceneNode>>
figma.setCanvasGrid(grid): void
figma.createCanvasRow(rowIndex?): SceneNode
figma.moveNodesToCoord(nodeIds, rowIndex?, columnIndex?): void
```

---

## 11. Draw APIs — Update 110 (May 2025 beta), Update 123 (Jan 2026)

```typescript
// Text on a path
figma.createTextPath(node: VectorNode, startSegment: number, startPosition: number): TextPathNode
// Supports all text styling: setRangeFontSize, setRangeFontName, insertCharacters, etc.

// Transform groups with modifiers
figma.transformGroup(nodes, parent, index, transformModifiers): TransformGroupNode

// Brush loading for stretch/scatter brushes
figma.loadBrushesAsync(brushType: 'STRETCH' | 'SCATTER'): Promise<void>

// Async fill/stroke setters (required for PatternPaint)
node.setFillsAsync(fills): Promise<void>
node.setStrokesAsync(strokes): Promise<void>

// Complex stroke properties
node.complexStrokeProperties: ComplexStrokeProperties
node.variableWidthStrokeProperties: VariableWidthStrokeProperties
```

---

## 12. JSX Node Creation

```typescript
figma.createNodeFromJSXAsync(jsx: any): Promise<SceneNode>
```

**Setup:** Install `@figma/widget-typings`. TSConfig: `"jsx": "react"`, `"jsxFactory": "figma.widget.h"`, `"jsxFragmentFactory": "figma.widget.Fragment"`. Rename to `.tsx`.

```typescript
const { AutoLayout, Image, Text, Frame, SVG, Ellipse, Rectangle, Line } = figma.widget
const node = await figma.createNodeFromJSXAsync(
  <AutoLayout fill="#F00" padding={20}>
    <Text fontSize={24} fill="#FFF">Hello</Text>
    <Image src="https://example.com/photo.jpg" width={200} height={200} />
  </AutoLayout>
)
```

Uses Widget API JSX elements. Create via JSX, then modify properties imperatively for full control.

---

## 13. Script Execution & Runtime

### Two-Thread Architecture

| | **Main Thread (Sandbox)** | **UI Thread (iframe)** |
|---|---|---|
| Runtime | QuickJS compiled to WASM | Standard browser iframe |
| Figma scene access | Full read/write (`figma.*` API) | None |
| Browser APIs | None (no DOM, no fetch) | Full (DOM, fetch, WebSocket) |
| Communication | `figma.ui.postMessage()` | `parent.postMessage({ pluginMessage: ... }, '*')` |

QuickJS is a C-based JS interpreter cross-compiled to WASM — secure by design, no JIT. The WASM sandbox has no access to browser APIs; plugin code can only reach the outside world through explicit whitelisted APIs.

### Main Thread — Available JS Features

**Available**: All ES6+ built-in types (`Object`, `Array`, `Map`, `Set`, `WeakMap`, `WeakSet`, `Symbol`, `Proxy`, `Reflect`), `Promise`/`async`/`await`, `JSON`, `Math`, `Date`, `RegExp`, `Uint8Array`/`ArrayBuffer`/`DataView`, `console.log`/`warn`/`error`, `setTimeout`/`setInterval`/`clearTimeout`/`clearInterval`, template literals, destructuring, spread, classes, generators, iterators.

**NOT available**: `document`, DOM, `window`, `fetch`, `XMLHttpRequest`, `WebSocket`, `localStorage`, `Image`, `Canvas`, `Worker`, `SharedArrayBuffer`, `eval()`.

### UI Thread — iframe Restrictions

- `Origin: null` (sandboxed) — `fetch()` only works against APIs returning `Access-Control-Allow-Origin: *`
- Manifest `networkAccess.allowedDomains` enforced via CSP — requests to unlisted domains throw CSP errors
- No `localStorage` — use `figma.clientStorage` from the main thread instead
- Full DOM, Canvas, SVG, `crypto`, `WebSocket`, `TextEncoder`/`TextDecoder` available

### Inter-Thread Communication (postMessage)

```typescript
// Main → UI
figma.ui.postMessage({ type: 'myEvent', payload: data })

// UI → Main
parent.postMessage({ pluginMessage: { type: 'myEvent', payload: data } }, '*')

// Main listening
figma.ui.onmessage = (msg) => { ... }
// or event-based:
figma.ui.on('message', (msg) => { ... })

// UI listening
window.onmessage = (event) => {
  const msg = event.data?.pluginMessage
  if (!msg) return
  // handle msg
}
```

Serialization: objects, arrays, primitives, `Date`, `Uint8Array` work. Functions, class instances (prototype chains stripped), `Map`/`Set` do NOT — serialize manually.

**Network request pattern**: Main thread sends request message to UI → UI performs `fetch()` → UI posts result back. Use a timeout guard to prevent hanging.

### figma.showUI()

```typescript
figma.showUI(html: string, options?: ShowUIOptions): void
```

| Option | Type | Default | Description |
|---|---|---|---|
| `width` | `number` | 300 | Plugin window width (min: 70) |
| `height` | `number` | 200 | Plugin window height (min: 0) |
| `visible` | `boolean` | `true` | Initially visible. Toggle with `figma.ui.show()`/`hide()` |
| `title` | `string` | plugin name | Window header title |
| `position` | `{ x, y }` | auto | Position on canvas (clamped to viewport) |
| `themeColors` | `boolean` | `false` | Injects CSS vars (`--figma-color-bg`, etc.) + `figma-light`/`figma-dark` class |

Related: `figma.ui.resize(w, h)`, `figma.ui.reposition(x, y)`, `figma.ui.close()` (closes UI only, not plugin).

### figma.notify()

```typescript
figma.notify(message: string, options?: NotificationOptions): NotificationHandler
```

| Option | Type | Default | Description |
|---|---|---|---|
| `timeout` | `number` | ~3000 | Duration ms. `Infinity` for indefinite |
| `error` | `boolean` | `false` | Red/error styling |
| `button` | `{ text, action }` | — | Action button on notification |
| `onDequeue` | `(reason) => void` | — | Fired when notification removed |

Returns `NotificationHandler` with `.cancel()`. Message limit: 100 characters.

### figma.closePlugin()

```typescript
figma.closePlugin(message?: string): void
```

- Optional `message` displayed as toast after closing
- Disables all Figma API callbacks but does NOT halt JS execution — always follow with `return`
- All `setTimeout`/`setInterval` timers are cancelled
- `figma.on('close', callback)` fires for cleanup

### Plugin Modes
```typescript
figma.editorType: 'figma' | 'figjam' | 'dev' | 'slides' | 'buzz'
figma.mode: 'default' | 'textreview' | 'inspect' | 'codegen' | 'linkpreview' | 'auth'
```

### Execution Limits

- **No hard execution timeout** for general plugin code. Long sync work freezes the Figma UI → "Plugin is not responding" dialog.
- **Codegen callbacks**: 15-second timeout (was 3s, extended April 2025).
- **Memory**: No documented limit. QuickJS WASM has finite memory — "RuntimeError: memory access out of bounds" on very large operations.
- **Frozen plugin mitigation**: Yield between chunks:
  ```typescript
  async function processNodes(nodes: SceneNode[]) {
    for (let i = 0; i < nodes.length; i++) {
      doWork(nodes[i])
      if (i % 100 === 0) await new Promise(r => setTimeout(r, 0)) // yield
    }
  }
  ```

### Timer API (FigJam)
```typescript
figma.timer?.start()
figma.timer?.stop()
figma.timer?.pause()
figma.timer?.resume()
// Events: 'timerstart', 'timerstop', 'timerpause', 'timerresume', 'timeradjust', 'timerdone'
```

### Parameters API
```typescript
figma.parameters.on('input', (event: ParameterInputEvent) => void)
```
For plugin commands with parameter UI.

### Plugin Manifest (manifest.json) — Complete Specification

```typescript
interface PluginManifest {
  // ─── Required ───
  name: string                        // Display name
  id: string                          // Unique plugin ID (assigned by Figma)
  api: string                         // API version, currently "1.0.0"
  editorType: EditorType[]            // Target editors
  main: string                        // Path to main JS entry (sandbox code)

  // ─── Optional ───
  ui?: string | Record<string, string>  // HTML for iframe UI
  build?: string                        // Shell command run before loading (e.g., "npm run build")
  documentAccess?: "dynamic-page"       // Required for new plugins since April 2024
  networkAccess?: NetworkAccess         // Domain allowlist for fetch/WebSocket
  permissions?: PermissionType[]        // API permissions
  menu?: ManifestMenuItem[]             // Multi-command menu (mutually exclusive with top-level parameters)
  parameters?: ParameterDefinition[]    // Quick Actions parameters (single-command plugins)
  parameterOnly?: boolean               // Only run when parameters provided (default: true when parameters set)
  relaunchButtons?: RelaunchButton[]    // Buttons on nodes via setRelaunchData()
  capabilities?: ("codegen" | "inspect" | "vscode")[]  // Dev Mode capabilities
  codegenLanguages?: { label: string, value: string }[] // Required when capabilities includes "codegen"
  codegenPreferences?: CodegenPreference[]              // Inspect panel UI preferences
  enableProposedApi?: boolean           // Enable experimental APIs (dev only, ignored in published)
  enablePrivatePluginApi?: boolean      // Enable private APIs (org-private plugins only)
}

type EditorType = 'figma' | 'figjam' | 'dev' | 'slides'  // 'buzz' exists internally

type PermissionType = 'currentuser' | 'activeusers' | 'fileusers' | 'payments' | 'teamlibrary'
```

**Network Access:**
```typescript
interface NetworkAccess {
  allowedDomains: string[]    // ["none"] | ["*"] | ["example.com", "*.cdn.example.com"]
  devAllowedDomains?: string[] // e.g. ["http://localhost:3000"]
  reasoning?: string           // Required when allowedDomains includes "*"
}
// Patterns: "example.com", "*.example.com", "https://example.com", "ws://localhost:3000"
```

**Menu Structure:**
```typescript
type ManifestMenuItem =
  | { name: string; command: string; parameters?: ParameterDefinition[]; parameterOnly?: boolean }
  | { separator: true }
  | { name: string; menu: ManifestMenuItem[] }  // nested submenu (recursive)

interface ParameterDefinition {
  name: string             // Display name in Quick Actions
  key: string              // Key in ParameterValues
  description?: string
  allowFreeform?: boolean  // Allow arbitrary text (default: false, only suggestions)
  optional?: boolean       // Can skip (default: false)
}
```

**Relaunch Buttons:**
```typescript
interface RelaunchButton {
  command: string              // Matches node.setRelaunchData({ [command]: description })
  name: string                 // Button text in right sidebar
  multipleSelection?: boolean  // Show when multiple nodes selected (default: false)
}
```

**Codegen Preferences:**
```typescript
type CodegenPreference =
  | { itemType: "unit"; scaledUnit: string; defaultScaleFactor: number; default?: boolean }
  | { itemType: "select"; propertyName: string; options: { label: string; value: string; isDefault?: boolean }[] }
  | { itemType: "action"; propertyName: string; label: string }
// All types accept optional includedLanguages?: string[]
```

**UI field:** A string → single HTML file (contents available as `__html__`). An object `{ key: path }` → named UI pages via `figma.showUI(__uiFiles__[key])`.

**Key gotchas:**
- `documentAccess: "dynamic-page"` required for new plugins — sync page APIs throw without it
- `menu` and top-level `parameters` are mutually exclusive
- `capabilities` requires `editorType` to include `"dev"`
- `enableProposedApi` is ignored in published plugins

### Events (Complete List)
```typescript
// No-argument events
'selectionchange' | 'currentpagechange' | 'close'
'timerstart' | 'timerstop' | 'timerpause' | 'timerresume' | 'timeradjust' | 'timerdone'

// Events with arguments
'run'               // RunEvent
'drop'              // DropEvent → boolean
'documentchange'    // DocumentChangeEvent
'slidesviewchange'  // SlidesViewChangeEvent
'canvasviewchange'  // CanvasViewChangeEvent
'textreview'        // TextReviewEvent → TextReviewRange[]
'stylechange'       // StyleChangeEvent

// figma.codegen.on()
'generate' | 'preferenceschange'

// figma.devResources.on()
'linkpreview' | 'auth' | 'open'

// figma.parameters.on()
'input'

// page.on()
'nodechange'

// figma.ui.on()
'message'
```

---

## 14. Data Storage

### Client Storage (per-plugin, per-user)
```typescript
figma.clientStorage.getAsync(key: string): Promise<any>
figma.clientStorage.setAsync(key: string, value: any): Promise<void>
figma.clientStorage.deleteAsync(key: string): Promise<void>
figma.clientStorage.keysAsync(): Promise<string[]>
```
Limits: 5 MB total, 100 KB per entry (Update 109, March 2025 — raised from 1 MB).

### Plugin Data (on nodes/document, shared across users)
```typescript
node.getPluginData(key: string): string
node.setPluginData(key: string, value: string): void
node.getPluginDataKeys(): string[]

// Shared between plugins (key = namespace)
node.getSharedPluginData(namespace, key): string
node.setSharedPluginData(namespace, key, value): void
node.getSharedPluginDataKeys(namespace): string[]
```

### Document-Level
```typescript
figma.root.getPluginData(key): string
figma.root.setPluginData(key, value): void
```

---

## 15. Dynamic Page Loading — Update 87 (Feb 2024)

Major architectural shift. Manifest: `"documentAccess": "dynamic-page"`.

```typescript
figma.loadAllPagesAsync(): Promise<void>
page.loadAsync(): Promise<void>
```

**13 sync methods deprecated** for async equivalents:
- `figma.getNodeById()` → `figma.getNodeByIdAsync()`
- `figma.getStyleById()` → `figma.getStyleByIdAsync()`
- `figma.getLocalPaintStyles()` → `figma.getLocalPaintStylesAsync()`
- (and 10 more — see Deprecations section)

Required for all new plugins since April 2024.

---

## 16. New Text Features

### Underline Customization — Update 106 (Dec 2024)
```typescript
textDecorationStyle: 'SOLID' | 'DOUBLE' | 'DOTTED' | 'DASHED' | 'WAVY'
textDecorationOffset: number
textDecorationThickness: number
textDecorationColor: RGBA
textDecorationSkipInk: boolean
```
Available as direct properties, via `getStyledTextSegments()`, and ranged getters/setters.

### Typography Variable Scopes — Updates 91-94 (April-May 2024)
New STRING scopes: `FONT_FAMILY`, `FONT_STYLE`, `TEXT_CONTENT`
New FLOAT scopes: `FONT_SIZE`, `FONT_WEIGHT`, `LINE_HEIGHT`, `LETTER_SPACING`, `PARAGRAPH_SPACING`, `PARAGRAPH_INDENT`

### Range-based Variable Binding
```typescript
node.getRangeBoundVariable(start, end, field): VariableAlias | null
node.setRangeBoundVariable(start, end, field, variable): void
```

### OpenType Features
```typescript
node.openTypeFeatures: { readonly [feature: string]: boolean }
```
Returns map of 4-character OT feature codes diverging from defaults.

---

## 17. New Prototyping Features

### Trigger Types (Complete)
```typescript
'ON_CLICK' | 'ON_HOVER' | 'ON_PRESS' | 'ON_DRAG' | 'AFTER_TIMEOUT'
| 'MOUSE_UP' | 'MOUSE_DOWN' | 'MOUSE_ENTER' | 'MOUSE_LEAVE'
| 'ON_KEY_DOWN'
| 'ON_MEDIA_HIT'    // NEW — media playback reaches timestamp
| 'ON_MEDIA_END'    // NEW — media playback ends
```

### Action Types (Complete)
```typescript
'BACK' | 'CLOSE' | 'URL' | 'NODE'
| 'UPDATE_MEDIA_RUNTIME'  // NEW — control media playback
| 'SET_VARIABLE'           // NEW — set variable value
| 'SET_VARIABLE_MODE'      // NEW — set variable collection mode
| 'CONDITIONAL'            // NEW — conditional logic
```

### Easing Types (Complete)
```typescript
'EASE_IN' | 'EASE_OUT' | 'EASE_IN_AND_OUT' | 'LINEAR'
| 'EASE_IN_BACK' | 'EASE_OUT_BACK' | 'EASE_IN_AND_OUT_BACK'
| 'CUSTOM_CUBIC_BEZIER'
| 'GENTLE' | 'QUICK' | 'BOUNCY' | 'SLOW'  // NEW spring presets
| 'CUSTOM_SPRING'                           // NEW
```

---

## 18. New Node Types Summary

| Node Type | Editor | Created Via | Update |
|---|---|---|---|
| TextPathNode | Figma | `createTextPath()` | 110 (May 2025) |
| TransformGroupNode | Figma | `transformGroup()` | 110 (May 2025) |
| SlideNode | Slides | `createSlide()` | 108 (Feb 2025) |
| SlideRowNode | Slides | `createSlideRow()` | 108 (Feb 2025) |
| SlideGridNode | Slides | auto-created | 108 (Feb 2025) |
| InteractiveSlideElementNode | Slides | not creatable | 108 (Feb 2025) |
| HighlightNode | FigJam | — | — |
| WashiTapeNode | FigJam | — | — |

### Complete SceneNode Union (33 types)
BooleanOperationNode, CodeBlockNode, ComponentNode, ComponentSetNode, ConnectorNode, EllipseNode, EmbedNode, FrameNode, GroupNode, HighlightNode, InstanceNode, InteractiveSlideElementNode, LineNode, LinkUnfurlNode, MediaNode, PolygonNode, RectangleNode, SectionNode, ShapeWithTextNode, SliceNode, SlideGridNode, SlideNode, SlideRowNode, StampNode, StarNode, StickyNode, TableNode, TextNode, TextPathNode, TransformGroupNode, VectorNode, WashiTapeNode, WidgetNode

### New Creation Methods
```typescript
figma.createTextPath(node, startSegment, startPosition): TextPathNode
figma.createPageDivider(dividerName?): PageNode
figma.createSlide(row?, col?): SlideNode
figma.createSlideRow(row?): SlideRowNode
figma.createCanvasRow(rowIndex?): SceneNode
figma.transformGroup(nodes, parent, index, modifiers): TransformGroupNode
figma.loadBrushesAsync(brushType): Promise<void>
```

### New StrokeCap Values — Update 114 (June 2025)
`'DIAMOND_FILLED'`, `'TRIANGLE_FILLED'`, `'CIRCLE_FILLED'`

---

## 19. Deprecations & Breaking Changes

| Date | Update | Change |
|---|---|---|
| Feb 2024 | 87 | 13 sync methods deprecated → async (required for new plugins by April 2024) |
| Feb 2025 | 107 | `constrainProportions` → `targetAspectRatio` / `lockAspectRatio()` / `unlockAspectRatio()` |
| Nov 2025 | 120 | `InstanceNode.resetOverrides()` → `removeOverrides()` |
| Nov 2025 | 120 | GridTrackSize setter no longer auto-converts FLEX → FIXED |
| REST API Apr 2025 | — | `files:read` scope → `file_content:read`, `file_metadata:read`, etc. |
| Plugin API | — | `textAutoResize: 'TRUNCATE'` deprecated → use `textTruncation: 'ENDING'` + `maxLines` instead |
| REST API Apr 2025 | — | PAT max expiry reduced to 90 days; non-expiring tokens removed |
| REST API Dec 2024 | — | HTTPS enforced on api.figma.com (no more HTTP) |
| REST API Oct 2024 | — | Library Analytics: `num_instances` → `usages`, `num_teams_using` → `teams_using` |
| Early (pre-2023) | — | `horizontalPadding` → `paddingLeft` + `paddingRight` |
| Early (pre-2023) | — | `verticalPadding` → `paddingTop` + `paddingBottom` |

### Deprecated Sync → Async Migration

```typescript
// Old (deprecated)              // New (required)
figma.getNodeById(id)            figma.getNodeByIdAsync(id)
figma.getStyleById(id)           figma.getStyleByIdAsync(id)
figma.getLocalPaintStyles()      figma.getLocalPaintStylesAsync()
figma.getLocalTextStyles()       figma.getLocalTextStylesAsync()
figma.getLocalEffectStyles()     figma.getLocalEffectStylesAsync()
figma.getLocalGridStyles()       figma.getLocalGridStylesAsync()
figma.currentPage = page         figma.setCurrentPageAsync(page)
node.fillStyleId = id            node.setFillStyleIdAsync(id)
node.strokeStyleId = id          node.setStrokeStyleIdAsync(id)
node.effectStyleId = id          node.setEffectStyleIdAsync(id)
node.gridStyleId = id            node.setGridStyleIdAsync(id)
node.reactions = [...]           node.setReactionsAsync([...])
node.vectorNetwork = {...}       node.setVectorNetworkAsync({...})
instance.mainComponent           instance.getMainComponentAsync()
component.instances              component.getInstancesAsync()
style.consumers                  style.getStyleConsumersAsync()
```

---

## 20. REST API — Complete Reference

Base URL: `https://api.figma.com`

### Complete Endpoint Catalog (48 endpoints)

**Files (5)**

| Method | Path | Description | Tier |
|--------|------|-------------|------|
| GET | `/v1/files/:key` | File JSON (document tree, metadata, nodes) | 1 |
| GET | `/v1/files/:key/nodes` | Specific nodes by IDs | 1 |
| GET | `/v1/images/:key` | Render/export images (JPG, PNG, SVG, PDF) | 1 |
| GET | `/v1/files/:key/images` | Download links for all image fills | 2 |
| GET | `/v1/files/:key/meta` | File metadata without content | 3 |

**Comments (6)**

| Method | Path | Description | Tier |
|--------|------|-------------|------|
| GET | `/v1/files/:key/comments` | List comments | 2 |
| POST | `/v1/files/:key/comments` | Post comment | 2 |
| DELETE | `/v1/files/:key/comments/:id` | Delete comment | 2 |
| GET | `/v1/files/:key/comments/:id/reactions` | List reactions | 2 |
| POST | `/v1/files/:key/comments/:id/reactions` | Add reaction | 2 |
| DELETE | `/v1/files/:key/comments/:id/reactions` | Delete reaction | 2 |

**Components & Component Sets (6)**

| Method | Path | Description | Tier |
|--------|------|-------------|------|
| GET | `/v1/teams/:id/components` | Team published components | 3 |
| GET | `/v1/files/:key/components` | File published components | 3 |
| GET | `/v1/components/:key` | Single component metadata | 3 |
| GET | `/v1/teams/:id/component_sets` | Team component sets | 3 |
| GET | `/v1/files/:key/component_sets` | File component sets | 3 |
| GET | `/v1/component_sets/:key` | Single component set | 3 |

**Styles (3)** — GET `/v1/teams/:id/styles`, `/v1/files/:key/styles`, `/v1/styles/:key` (all Tier 3)

**Variables (3, Enterprise)**

| Method | Path | Description | Tier |
|--------|------|-------------|------|
| GET | `/v1/files/:key/variables/local` | Local + remote variables | 2 |
| GET | `/v1/files/:key/variables/published` | Published variables | 2 |
| POST | `/v1/files/:key/variables` | Bulk create/update/delete | 3 |

**Other**: Version History (1, Tier 2), Projects (2, Tier 2), Users (1, Tier 3), Dev Resources (4, Tier 2), Webhooks v2 (7, Tier 2), Library Analytics (6, Enterprise Tier 3), Activity Logs (1, Enterprise Tier 3), Discovery (1, Enterprise+Governance Tier 2), Payments (2)

### Rate Limits by Plan (Leaky-Bucket, Per-User)

| Tier | View/Collab | Starter Dev/Full | Pro Dev/Full | Org Dev/Full |
|------|-------------|-------------------|--------------|--------------|
| **1** | 6/month | 10/min | 15/min | 20/min |
| **2** | 5/min | 25/min | 50/min | 100/min |
| **3** | 10/min | 50/min | 100/min | 150/min |

429 response headers: `Retry-After`, `X-Figma-Plan-Tier`, `X-Figma-Rate-Limit-Type` (low/high).

### Authentication

**PAT**: Header `X-Figma-Token: <token>`. Max expiry 90 days (non-expiring removed April 2025).

**OAuth 2.0** (Authorization Code + PKCE):
```
GET https://www.figma.com/oauth?client_id=...&redirect_uri=...&scope=...&state=...&response_type=code
POST https://api.figma.com/v1/oauth/token  (exchange code)
POST https://api.figma.com/v1/oauth/refresh (refresh token)
```
Header: `Authorization: Bearer <access_token>`. Token expiry: 90 days. Auth codes expire in 30 seconds.

### OAuth Scopes (21 total)

| Scope | Description | Plan |
|-------|-------------|------|
| `current_user:read` | Name, email, profile image | — |
| `file_content:read` | File contents (nodes, editor type) | — |
| `file_metadata:read` | File metadata | — |
| `file_comments:read` / `write` | Read/write comments and reactions | — |
| `file_dev_resources:read` / `write` | Read/write dev resources | — |
| `file_variables:read` / `write` | Read/write variables | Enterprise |
| `file_versions:read` | Version history | — |
| `library_analytics:read` | Design system analytics | Enterprise |
| `library_assets:read` | Individual published component/style data | — |
| `library_content:read` | Published components/styles in files | — |
| `team_library_content:read` | Published components/styles in teams | — |
| `projects:read` | List projects and files | — |
| `selections:read` | Most recent file selection | — |
| `webhooks:read` / `write` | Read/manage webhooks | — |
| `org:activity_log_read` | Organization activity logs | Enterprise admin |
| `org:discovery_read` | Text event data | Enterprise+Governance admin |
| `files:read` | **DEPRECATED** — broad access (replaced by granular scopes) | — |

### Variables REST API — Deep Dive

**GET response structure** (`/v1/files/:key/variables/local`):
```json
{
  "meta": {
    "variables": {
      "<variableId>": {
        "id": "VariableID:123:456",
        "name": "colors/primary",
        "key": "abc123...",
        "variableCollectionId": "VariableCollectionID:123:0",
        "resolvedType": "COLOR",         // BOOLEAN | FLOAT | STRING | COLOR
        "valuesByMode": {
          "<modeId>": { "r": 0.2, "g": 0.4, "b": 0.8, "a": 1 }
          // or: true | 16 | "hello"
          // or: { "type": "VARIABLE_ALIAS", "id": "VariableID:..." }
        },
        "scopes": ["ALL_SCOPES"],
        "codeSyntax": { "WEB": "--color-primary", "ANDROID": "colorPrimary", "iOS": "colorPrimary" }
      }
    },
    "variableCollections": {
      "<collectionId>": {
        "id": "VariableCollectionID:123:0",
        "name": "Colors",
        "modes": [{ "modeId": "123:0", "name": "Light" }, { "modeId": "123:1", "name": "Dark" }],
        "defaultModeId": "123:0",
        "variableIds": ["VariableID:123:456"]
      }
    }
  }
}
```

**POST request** (`/v1/files/:key/variables`) — all four arrays optional, atomic bulk:
```json
{
  "variableCollections": [{ "action": "CREATE", "id": "temp_1", "name": "Spacing" }],
  "variableModes": [{ "action": "CREATE", "id": "temp_m1", "name": "Default", "variableCollectionId": "temp_1" }],
  "variables": [{ "action": "CREATE", "id": "temp_v1", "name": "spacing/sm", "variableCollectionId": "temp_1", "resolvedType": "FLOAT", "scopes": ["GAP", "WIDTH_HEIGHT"] }],
  "variableModeValues": [{ "variableId": "temp_v1", "modeId": "temp_m1", "value": 8 }]
}
```
Response maps temp IDs to real Figma IDs: `{ "meta": { "tempIdToRealId": { "temp_1": "VariableCollectionID:789:0", ... } } }`

Actions: `CREATE` (use temp ID) | `UPDATE` (use real ID) | `DELETE` (use real ID, no other fields needed).

Extended collections (Nov 2025): `parentVariableCollectionId`, `isExtension`, `variableOverrides`.

### New REST API Node Properties (2024-2025)
- Grid: `gridRowCount`, `gridColumnCount`, `gridRowGap`, `gridColumnGap`, `gridRowsSizing`, `gridColumnsSizing`
- Effects: `TEXTURE`, `NOISE`, `GLASS` types; `PROGRESSIVE` blur
- Paints: `PATTERN` type
- Nodes: `TEXT_PATH`, `TRANSFORM_GROUP` types
- Text: `fontStyle`, `semanticWeight`, `semanticItalic`
- Misc: `targetAspectRatio`, `complexStrokeProperties`, `variableWidthPoints`

---

## 21. Webhooks

### Event Types
- `FILE_UPDATE` — file content changes
- `FILE_VERSION_UPDATE` — new version saved
- `FILE_COMMENT` — comment added/modified
- `FILE_DELETE` — file deleted
- `LIBRARY_PUBLISH` — library published
- `DEV_MODE_STATUS_UPDATE` — Dev Mode status changes (NEW, May 2025)

### Context Types
Team, Project, or File level.

### Limits
| Context | Limit |
|---|---|
| Team | 20 webhooks |
| Project | 5 |
| File | 3 |
| Plan total | Pro: 150, Org: 300, Enterprise: 600 |

Retry: 3 retries at 5min, 30min, 3hr intervals.

---

## Complete `figma.*` Top-Level API

### Properties
`apiVersion`, `command`, `editorType`, `mode`, `pluginId`, `widgetId`, `fileKey`, `skipInvisibleInstanceChildren`, `root`, `currentPage`, `hasMissingFont`

### Sub-APIs
`timer`, `viewport`, `currentUser`, `activeUsers`, `textreview`, `codegen`, `vscode`, `devResources`, `payments`, `ui`, `util`, `constants`, `clientStorage`, `parameters`, `variables`, `teamLibrary`, `annotations`, `buzz`

### Utility Methods
```typescript
figma.util.rgb(color: string | RGB): RGB
figma.util.rgba(color: string | RGB): RGBA
figma.util.solidPaint(color, overrides?): SolidPaint
figma.util.normalizeMarkdown(markdown): string  // NEW
```

### Constants
```typescript
figma.constants.colors  // ColorPalettes — predefined Figma color palettes
```

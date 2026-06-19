# New Figma Effects & Paint Types (2025+)

Complete reference for effect and paint types added to the Figma Plugin API in 2025. All new effects are currently **beta** — API may change.

**Updated Effect union:**
```typescript
type Effect = DropShadowEffect | InnerShadowEffect | BlurEffect | NoiseEffect | TextureEffect | GlassEffect
```

**Updated Paint union:**
```typescript
type Paint = SolidPaint | GradientPaint | ImagePaint | VideoPaint | PatternPaint
```

---

## Effect Limits Per Layer

| Effect Type | Max Per Layer | Notes |
|---|---|---|
| DROP_SHADOW | 8 | |
| INNER_SHADOW | 8 | |
| LAYER_BLUR | 1 | Cannot combine with BACKGROUND_BLUR |
| BACKGROUND_BLUR | 1 | Cannot combine with GLASS |
| NOISE | 2 | |
| TEXTURE | 1 | |
| GLASS | 1 | Cannot combine with BACKGROUND_BLUR |

---

## 1. Glass Effect (`type: 'GLASS'`) — Update 116, July 2025

Translucent refractive glass simulation with specular highlights and chromatic aberration.

```typescript
interface GlassEffect {
  readonly type: 'GLASS'
  readonly visible: boolean
  readonly lightIntensity: number  // 0-1, brightness of specular highlights
  readonly lightAngle: number      // degrees, direction of light source
  readonly refraction: number      // 0-1, optical distortion along curved edge
  readonly depth: number           // >= 1, how far curved edge extends inward
  readonly dispersion: number      // 0-1, chromatic aberration (rainbow splitting)
  readonly radius: number          // frost/background blur amount
  readonly boundVariables?: {}     // variable binding not yet supported
}
```

### Code Example

```typescript
const frame = figma.createFrame()
frame.resize(300, 200)
// IMPORTANT: Glass is NOT visible on 100% opaque fills — reduce opacity
frame.fills = [{ type: 'SOLID', color: { r: 1, g: 1, b: 1 }, opacity: 0.3 }]

frame.effects = [{
  type: 'GLASS',
  visible: true,
  lightIntensity: 0.5,
  lightAngle: 45,
  refraction: 0.3,
  depth: 2,
  dispersion: 0.1,
  radius: 10,
}]
```

### Gotchas
- Fill opacity must be < 100% for glass to be visible
- Cannot combine with BACKGROUND_BLUR on same node
- Initially Frame-only in API; UI supports any object since Jan 2026
- `splay` property visible in UI but NOT in Plugin API yet
- Variable binding not supported
- Light reflection border may not render when applied via Plugin API (known bug)
- Cannot export to SVG — raster export only

---

## 2. Progressive Blur (`blurType: 'PROGRESSIVE'`) — Update 110, May 2025

Blur that transitions from sharp to blurry across a gradient direction.

```typescript
type BlurEffect = BlurEffectNormal | BlurEffectProgressive

interface BlurEffectBase {
  readonly type: 'LAYER_BLUR' | 'BACKGROUND_BLUR'
  readonly radius: number
  readonly visible: boolean
  readonly boundVariables?: { ['radius']?: VariableAlias }
}

interface BlurEffectNormal extends BlurEffectBase {
  readonly blurType: 'NORMAL'
}

interface BlurEffectProgressive extends BlurEffectBase {
  readonly blurType: 'PROGRESSIVE'
  readonly startRadius: number      // blur radius at start point
  readonly startOffset: Vector      // normalized 0-1 object space (top-left = 0,0)
  readonly endOffset: Vector        // normalized 0-1 object space (bottom-right = 1,1)
}
```

### Code Example

```typescript
const frame = figma.createFrame()
frame.resize(400, 300)
frame.fills = [{ type: 'SOLID', color: { r: 0.2, g: 0.4, b: 0.8 } }]

frame.effects = [{
  type: 'BACKGROUND_BLUR',
  blurType: 'PROGRESSIVE',
  radius: 20,                          // end radius (fully blurred)
  startRadius: 0,                      // start radius (sharp)
  startOffset: { x: 0.5, y: 0 },      // top center
  endOffset: { x: 0.5, y: 1 },        // bottom center
  visible: true,
}]
```

### Gotchas
- Coordinates use normalized object space: (0,0) top-left, (1,1) bottom-right
- Blur transitions from `startRadius` at `startOffset` to `radius` at `endOffset`
- Works with both LAYER_BLUR and BACKGROUND_BLUR
- For background blur, fill opacity must be between 0.10% and 99.99%
- Old code that assumes all blur effects are normal will break — check `blurType`

---

## 3. Noise Effect (`type: 'NOISE'`) — Update 110, May 2025

Random pixel grain with three sub-types.

```typescript
type NoiseEffect = NoiseEffectMonotone | NoiseEffectDuotone | NoiseEffectMultitone

interface NoiseEffectBase {
  readonly type: 'NOISE'
  readonly visible: boolean
  readonly blendMode: BlendMode
  readonly noiseSize: number    // scale of noise pixels
  readonly density: number      // concentration of noise pixels
  readonly color: RGBA          // primary noise color (added Update 114)
  readonly boundVariables?: {}  // not yet functional
}

interface NoiseEffectMonotone extends NoiseEffectBase {
  readonly noiseType: 'MONOTONE'    // single color
}

interface NoiseEffectDuotone extends NoiseEffectBase {
  readonly noiseType: 'DUOTONE'     // two colors
  readonly secondaryColor: RGBA
}

interface NoiseEffectMultitone extends NoiseEffectBase {
  readonly noiseType: 'MULTITONE'   // many colors
  readonly opacity: number
}
```

### Code Example

```typescript
const rect = figma.createRectangle()
rect.resize(400, 300)
rect.fills = [{ type: 'SOLID', color: { r: 0.95, g: 0.93, b: 0.9 } }]

// Monotone grain
rect.effects = [{
  type: 'NOISE',
  noiseType: 'MONOTONE',
  visible: true,
  blendMode: 'NORMAL',
  noiseSize: 1.5,
  density: 0.5,
  color: { r: 0, g: 0, b: 0, a: 0.15 },
}]

// Duotone grain
rect.effects = [{
  type: 'NOISE',
  noiseType: 'DUOTONE',
  visible: true,
  blendMode: 'NORMAL',
  noiseSize: 2,
  density: 0.4,
  color: { r: 0.8, g: 0.2, b: 0.1, a: 0.3 },
  secondaryColor: { r: 0.1, g: 0.2, b: 0.8, a: 0.3 },
}]
```

### Gotchas
- Max 2 noise effects per object
- `color` property added in Update 114 (June 2025)
- `visible` and `boundVariables` added in Update 116 (July 2025)

---

## 4. Texture Effect (`type: 'TEXTURE'`) — Update 110, May 2025

Roughened/textured edge effect.

```typescript
interface TextureEffect {
  readonly type: 'TEXTURE'
  readonly visible: boolean
  readonly noiseSize: number       // scale of textured effect
  readonly radius: number          // how far past the layer boundary the effect spreads
  readonly clipToShape: boolean    // limits texture within layer boundary
  readonly boundVariables?: {}     // not yet functional
}
```

### Code Example

```typescript
const rect = figma.createRectangle()
rect.resize(200, 200)
rect.fills = [{ type: 'SOLID', color: { r: 0.3, g: 0.3, b: 0.3 } }]

rect.effects = [{
  type: 'TEXTURE',
  visible: true,
  noiseSize: 3,
  radius: 5,
  clipToShape: false,   // texture extends past boundary
}]
```

### Gotchas
- Max 1 texture effect per object
- Affects edges of the object, giving roughened appearance

---

## 5. Pattern Paint (`type: 'PATTERN'`) — Update 110, May 2025

Tile-based pattern fill from a source node.

```typescript
interface PatternPaint {
  readonly type: 'PATTERN'
  readonly sourceNodeId: string
  readonly tileType: 'RECTANGULAR' | 'HORIZONTAL_HEXAGONAL' | 'VERTICAL_HEXAGONAL'
  readonly scalingFactor: number
  readonly spacing: Vector              // { x: number, y: number }
  readonly horizontalAlignment: 'START' | 'CENTER' | 'END'
  readonly visible?: boolean
  readonly opacity?: number
  readonly blendMode?: BlendMode
}
```

### Code Example

```typescript
// Create pattern source
const patternSource = figma.createStar()
patternSource.resize(20, 20)
patternSource.fills = [{ type: 'SOLID', color: { r: 1, g: 0.4, b: 0 } }]

// Apply as pattern fill — MUST use async setter
const target = figma.createRectangle()
target.resize(400, 300)
await target.setFillsAsync([{
  type: 'PATTERN',
  sourceNodeId: patternSource.id,
  tileType: 'RECTANGULAR',
  scalingFactor: 1,
  spacing: { x: 5, y: 5 },
  horizontalAlignment: 'CENTER',
  visible: true,
  opacity: 1,
  blendMode: 'NORMAL',
}])
```

### Gotchas
- **Must use `setFillsAsync()` / `setStrokesAsync()`** — direct `node.fills = [...]` throws error
- `visible`, `opacity`, `blendMode` properties added in Update 113

---

## Timeline

| Date | Update | Change |
|---|---|---|
| May 7, 2025 | 110 | Noise, Texture, Progressive Blur, Pattern Paint (all beta) |
| May 22, 2025 | 113 | Pattern Paint: visible, opacity, blendMode properties |
| June 13, 2025 | 114 | Noise: `color` property |
| July 17, 2025 | 116 | Glass effect (beta); Noise & Texture: `visible` + `boundVariables` |
| ~Jan 2026 | UI | Glass out of beta in UI; splay, variable support, any-object in UI |

# Figma Design Workflows for AI Agents

## Table of Contents
1. [Creating a Page from Scratch](#1-creating-a-page-from-scratch)
2. [Editing an Existing Design](#2-editing-an-existing-design)
3. [Extracting a Design System](#3-extracting-a-design-system)
4. [Building Components](#4-building-components)
5. [Figma-to-Code](#5-figma-to-code)
6. [Common Recipes](#6-common-recipes)

---

## 1. Creating a Page from Scratch

### Prerequisites
- Write-capable MCP server (figma-console-mcp or similar)
- Figma Desktop app running with companion plugin active

### Step-by-step

**1. Create the artboard (top-level frame)**
```
Create a frame:
- Desktop: 1440 x 900
- Tablet: 768 x 1024
- Mobile: 390 x 844
Set fills to background color, e.g., [{type: 'SOLID', color: {r: 0.98, g: 0.98, b: 0.98}}]
```

**2. Set up auto layout on the root frame**
```
layoutMode: 'VERTICAL'
primaryAxisAlignItems: 'MIN'
counterAxisAlignItems: 'CENTER'
paddingTop/Right/Bottom/Left: 0
itemSpacing: 0
layoutSizingHorizontal: 'FIXED'
layoutSizingVertical: 'FIXED'
```

**3. Build the header section**
- Create a child frame (horizontal auto layout)
- Width: fill parent, height: hug content
- Add logo text/image on left, nav items on right
- Use `primaryAxisAlignItems: 'SPACE_BETWEEN'`

**4. Build the hero section**
- Create a child frame (vertical auto layout)
- Generous padding (80-120px vertical)
- Large heading text (48-72px), subtext (18-24px)
- CTA button(s)

**5. Build content sections one at a time**
- Each section is a vertical frame with padding
- Add content incrementally — one visual group per operation
- Use nested frames for grid layouts

**6. Review every 2-3 additions**
- Take screenshot
- Check spacing, alignment, contrast, typography hierarchy
- Fix issues before continuing

**7. Add footer**
- Horizontal frame at bottom
- Links, copyright, social icons

### Design Brief Template (prepare before building)
```
Color palette:
  - Background: #FAFAFA
  - Surface: #FFFFFF
  - Primary text: #1A1A1A
  - Secondary text: #6B6B6B
  - Accent: #2563EB
  - Accent hover: #1D4ED8

Typography:
  - Display: Inter Bold 48-72px
  - Heading: Inter Semibold 24-32px
  - Body: Inter Regular 16-18px
  - Caption: Inter Regular 14px
  - Line heights: 1.2 (display), 1.4 (heading), 1.6 (body)

Spacing rhythm:
  - Section gap: 80-120px
  - Group gap: 32-48px
  - Element gap: 12-16px
  - Component padding: 16-24px

Visual direction: Clean, minimal, generous whitespace
```

---

## 2. Editing an Existing Design

### Step-by-step

**1. Understand the current state**
- Get the selection or target node info
- Get tree summary to understand structure
- Take a screenshot for visual reference

**2. Identify what to change**
- Text content → use text content tools
- Colors/styles → update fills, strokes, effects
- Layout/spacing → modify auto layout properties
- Structure → add/remove/reorder nodes

**3. Make changes incrementally**
- One change at a time
- Screenshot after every 2-3 modifications
- Verify the change didn't break surrounding layout

### Common Edit Operations

**Change text:**
```
1. Load the font: figma.loadFontAsync({family, style})
2. Set node.characters = "New text"
```

**Change colors:**
```
node.fills = [{type: 'SOLID', color: {r: 0.15, g: 0.38, b: 0.92}}]
```

**Change spacing:**
```
frame.itemSpacing = 24     // gap between children
frame.paddingTop = 32      // internal padding
```

**Resize:**
```
node.resize(newWidth, newHeight)
// or for auto-layout children:
node.layoutSizingHorizontal = 'FILL'  // fill parent
node.layoutSizingHorizontal = 'HUG'   // fit content
```

**Reorder children:**
```
// Move child to specific index
parent.insertChild(newIndex, child)
```

**Add a new element:**
```
const rect = figma.createRectangle()
rect.resize(200, 48)
rect.cornerRadius = 8
rect.fills = [{type: 'SOLID', color: {r: 0.15, g: 0.38, b: 0.92}}]
parent.appendChild(rect)
```

---

## 3. Extracting a Design System

### From Figma Official MCP

**1. Get variables (design tokens)**
```
get_variable_defs → returns variable name → value mappings
Example: {'icon/default/secondary': '#949494', 'spacing/lg': 24}
```

**2. Generate design system rules**
```
create_design_system_rules → generates a rules file with:
- Layout primitives
- Component organization
- Naming conventions
- Token usage patterns
- Accessibility requirements
```

**3. Get component code**
```
get_design_context → returns React+Tailwind (or other framework) code
get_code_connect_map → maps Figma components to codebase components
```

### From Community MCP (figma-console-mcp)

**1. Extract all variables**
- Use variable management tools to read all collections, modes, and values
- Export as CSS custom properties, Tailwind config, or Sass variables

**2. Extract styles**
- Read all paint styles (colors)
- Read all text styles (typography)
- Read all effect styles (shadows, blurs)

**3. Extract components**
- List all components and component sets
- Read component properties (variants, slots)
- Export component structure

---

## 4. Building Components

### Simple Button Component

**1. Create the component**
```
const button = figma.createComponent()
button.name = "Button"
```

**2. Set up auto layout**
```
button.layoutMode = 'HORIZONTAL'
button.counterAxisAlignItems = 'CENTER'
button.primaryAxisAlignItems = 'CENTER'
button.paddingLeft = 24
button.paddingRight = 24
button.paddingTop = 12
button.paddingBottom = 12
button.itemSpacing = 8
button.cornerRadius = 8
```

**3. Add fills**
```
button.fills = [{type: 'SOLID', color: {r: 0.15, g: 0.38, b: 0.92}}]
```

**4. Add text label**
```
const label = figma.createText()
await figma.loadFontAsync({family: 'Inter', style: 'Medium'})
label.characters = 'Button'
label.fontSize = 16
label.fontName = {family: 'Inter', style: 'Medium'}
label.fills = [{type: 'SOLID', color: {r: 1, g: 1, b: 1}}]
button.appendChild(label)
```

**5. Add component properties**
```
button.addComponentProperty('label', 'TEXT', 'Button')
button.addComponentProperty('variant', 'VARIANT', 'primary')
```

### Creating Variants
```
// Create secondary variant
const secondary = button.clone()
secondary.name = "Button/secondary"
secondary.fills = [{type: 'SOLID', color: {r: 0.95, g: 0.95, b: 0.95}}]

// Combine into variant set
const buttonSet = figma.combineAsVariants([button, secondary], parent)
```

---

## 5. Figma-to-Code

### Using Official MCP

**Quick workflow:**
1. User shares Figma URL → extract node-id
2. `get_design_context(nodeId)` → get code + screenshot
3. Adapt the returned code to target framework
4. `get_variable_defs(nodeId)` → get design tokens for accurate values

**Advanced workflow:**
1. `get_metadata` on the page → understand full structure
2. `get_design_context` on each major section
3. `get_variable_defs` for consistent token usage
4. `get_code_connect_map` to use existing component mappings
5. Generate code that uses the project's actual components

### Tips for Better Code Output

- Always specify `artifactType` in `get_design_context` — it affects output format
- Use `get_metadata` first for complex pages, then drill into specific sections
- Check for existing Code Connect mappings before generating new code
- Use design system rules (`create_design_system_rules`) for consistent output

---

## 6. Common Recipes

### Add an image fill to a frame
```
const image = await figma.createImageAsync('https://example.com/photo.jpg')
frame.fills = [{type: 'IMAGE', imageHash: image.hash, scaleMode: 'FILL'}]
```

### Create a card with shadow
```
const card = figma.createFrame()
card.resize(320, 200)
card.cornerRadius = 12
card.fills = [{type: 'SOLID', color: {r: 1, g: 1, b: 1}}]
card.effects = [{
  type: 'DROP_SHADOW',
  color: {r: 0, g: 0, b: 0, a: 0.08},
  offset: {x: 0, y: 4},
  radius: 12,
  spread: 0,
  visible: true,
  blendMode: 'NORMAL'
}]
```

### Create a horizontal divider
```
const divider = figma.createLine()
divider.resize(parentWidth, 0)
divider.strokes = [{type: 'SOLID', color: {r: 0.9, g: 0.9, b: 0.9}}]
divider.strokeWeight = 1
```

### Create a gradient background
```
frame.fills = [{
  type: 'GRADIENT_LINEAR',
  gradientStops: [
    {position: 0, color: {r: 0.1, g: 0.1, b: 0.3, a: 1}},
    {position: 1, color: {r: 0.2, g: 0.1, b: 0.5, a: 1}}
  ],
  gradientTransform: [[0, 1, 0], [-1, 0, 1]]
}]
```

### Set up responsive text
```
text.textAutoResize = 'WIDTH_AND_HEIGHT'  // fits content exactly
// or
text.textAutoResize = 'HEIGHT'  // fixed width, grows vertically
text.resize(300, text.height)   // set fixed width
```

### Create an icon placeholder (circle with icon)
```
const circle = figma.createEllipse()
circle.resize(40, 40)
circle.fills = [{type: 'SOLID', color: {r: 0.95, g: 0.95, b: 0.95}}]

// Add an SVG icon inside
const icon = figma.createNodeFromSvg('<svg viewBox="0 0 24 24">...</svg>')
icon.resize(20, 20)
icon.x = 10
icon.y = 10
```

### Clone and modify (pattern repetition)
```
const clone = originalNode.clone()
clone.name = "Variant B"
// Modify specific properties
clone.fills = newFills
// Move to avoid overlap
clone.x = originalNode.x + originalNode.width + 40
```

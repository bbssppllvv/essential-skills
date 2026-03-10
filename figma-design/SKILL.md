---
name: figma-design
description: >
  Comprehensive guide for AI agents designing in Figma — reading designs, creating new UI,
  and editing existing designs programmatically. Covers the official Figma MCP server (13 tools,
  mostly read-only), the Figma Plugin API (full read/write, 37+ node types), the REST API
  (read-only for design data), and community MCP servers that enable full write capabilities
  via WebSocket + Plugin API bridge (up to 57+ tools).
  Use this skill whenever: working with Figma files or designs, doing Figma-to-code, creating
  or editing designs in Figma programmatically, setting up Figma MCP servers, building Figma
  plugins for AI-driven design, extracting design tokens or variables from Figma, comparing
  Figma capabilities with Paper or Pencil, or any task involving figma.com, Figma Desktop,
  Figma Plugin API, or Figma REST API. Also triggers on: mcp__figma-desktop__, figma MCP,
  design-to-code from Figma, Figma AI plugin, Figma design system, Figma variables.
---

# Figma Design — AI Agent Skill

This skill equips you to work with Figma as a design tool at a level comparable to Paper and Pencil MCP integrations. The core challenge: Figma's official MCP is primarily read-only for design nodes — you can inspect designs but not create or edit them directly. This skill teaches you the full landscape and the practical paths to full design capabilities.

## Architecture Overview

Figma exposes three API surfaces with very different capabilities:

| API Surface | Read | Write (design nodes) | Requires Figma open? |
|---|---|---|---|
| **REST API** | Full file tree, all properties | No (only comments, variables*, dev resources) | No |
| **Plugin API** | Full file tree, all properties | Yes — full CRUD on all 37+ node types | Yes (runs inside Figma) |
| **Official MCP** | Design context, screenshots, metadata, variables | Limited (`generate_figma_design` sends live UI) | Desktop: yes, Remote: no |

*Variables write requires Enterprise plan.

The **Plugin API** is the only path to full design editing power. Every community MCP server that offers write capabilities works by bridging the Plugin API to external tools via WebSocket.

## What Tools Are Available?

### If you have `mcp__figma-desktop__*` tools

You have Figma's **official desktop MCP**. 6 tools, all read-only:

- `get_design_context` — Primary tool. Returns code (React+Tailwind default) + screenshot + metadata for a node
- `get_metadata` — Sparse XML tree (IDs, types, names, positions, sizes)
- `get_screenshot` — Screenshot of a node
- `get_variable_defs` — Design token values (colors, spacing, typography)
- `get_figjam` — FigJam board content
- `create_design_system_rules` — Generates design system ruleset for agent context

These are powerful for **reading and understanding** designs. Use `get_design_context` as your primary inspection tool — it returns framework-specific code you can adapt.

### If you have community write tools (e.g., figma-console-mcp)

You have full read/write access. See `references/mcp-ecosystem.md` for the complete tool catalog. Key write operations:
- Create frames, rectangles, ellipses, text, vectors, components
- Modify fills, strokes, effects, auto layout, text content
- Manage variables, styles, and design tokens
- Boolean operations, grouping, hierarchy manipulation

### If you only have the REST API

You can read designs but **cannot create or edit design nodes**. You can:
- Read full file structure via `GET /v1/files/{key}`
- Render images via `GET /v1/images/{key}`
- Read/write variables (Enterprise only)
- Read/write comments and dev resources

## Design-to-Code Workflow (Read Path)

When the user wants to convert a Figma design to code:

1. **Get context**: Call `get_design_context` with the node ID (extract from Figma URL: `?node-id=X-Y` → nodeId `X:Y`)
2. **Check variables**: Call `get_variable_defs` for design tokens
3. **Get structure**: If the design is large, use `get_metadata` first for a sparse overview, then target specific nodes
4. **Screenshot**: Use `get_screenshot` for visual reference
5. **Generate code**: Adapt the returned reference code to the target framework

The returned code from `get_design_context` defaults to React + Tailwind but can be customized to Vue, HTML+CSS, iOS (SwiftUI), or Android (Jetpack Compose).

## Creating Designs in Figma (Write Path)

For creating or editing designs, you need write-capable tools — either through a community MCP server or by guiding the user to install one. See `references/mcp-ecosystem.md` for setup instructions.

### Design Creation Workflow

1. **Plan**: Define the design brief — color palette, typography, spacing rhythm, visual direction
2. **Create frame**: Start with a top-level frame (artboard) — desktop 1440x900, tablet 768x1024, mobile 390x844
3. **Build incrementally**: Add one visual group at a time (header, section, card, row) — never batch an entire screen
4. **Use auto layout**: Figma's auto layout maps to CSS Flexbox. Set `layoutMode: 'VERTICAL'` or `'HORIZONTAL'`, use `itemSpacing` for gaps, padding properties for internal space
5. **Screenshot and review**: Every 2-3 modifications, take a screenshot and evaluate spacing, typography, contrast, alignment, clipping
6. **Iterate**: Refine based on visual review

### Key Figma Concepts for AI Agents

**Frames are the primary container** — equivalent to `<div>` in HTML. They support auto layout, fills, strokes, effects, corner radius, and clipping.

**Auto Layout** maps directly to CSS Flexbox:
- `layoutMode: 'HORIZONTAL'` → `flex-direction: row`
- `layoutMode: 'VERTICAL'` → `flex-direction: column`
- `primaryAxisAlignItems: 'SPACE_BETWEEN'` → `justify-content: space-between`
- `counterAxisAlignItems: 'CENTER'` → `align-items: center`
- `layoutSizingHorizontal: 'FILL'` → `flex: 1` (fill parent)
- `layoutSizingHorizontal: 'HUG'` → `width: fit-content`

**Text requires font loading** — before setting `.characters`, `.fontSize`, or `.fontName`, you must call `figma.loadFontAsync()`. This is a common gotcha.

**Fills are arrays of Paint objects** — not simple colors. A solid red fill is:
```json
[{ "type": "SOLID", "color": { "r": 1, "g": 0, "b": 0 }, "opacity": 1 }]
```

**Components** create reusable design elements. A ComponentNode is a frame that can be instantiated. Variants are grouped in ComponentSetNodes.

**Variables** are design tokens — reusable values (COLOR, FLOAT, STRING, BOOLEAN) organized in collections with modes (light/dark, desktop/mobile).

## Quality Checklist (Review Every 2-3 Modifications)

- **Spacing**: Uneven gaps, cramped groups, areas that feel unintentionally empty. Is there visual rhythm?
- **Typography**: Text too small to read, poor line-height, weak hierarchy between heading/body/caption
- **Contrast**: Low contrast text, elements blending into background, overly uniform color
- **Alignment**: Elements that should share a vertical or horizontal lane but don't
- **Clipping**: Content cut off at container edges
- **Repetition**: Overly grid-like sameness — vary scale, weight, or spacing for visual interest

## Design Quality Principles

These principles apply regardless of which tools you're using:

- Be a minimalist: fewer, more refined elements. White space is a feature.
- Vary spacing deliberately — tighter to group related elements, generous to let hero content breathe
- Invest in text hierarchy and contrast. Pair heavy display type with light labels.
- One intense color moment is stronger than five. Build palettes from neutrals first.
- Default body text should never be pure black or pure gray — calibrate to palette warmth.
- Text contrast is non-negotiable. Muted text is useful for hierarchy but must remain legible.
- Use realistic placeholder content, not lorem ipsum.

## Reference Files

Read these for deep dives on specific topics:

- `references/plugin-api.md` — Full Plugin API capabilities: node types, properties, creation methods, auto layout, variables, components, text, vectors, effects, styles, events. Read when building a Plugin or using write-capable MCP tools that expose Plugin API operations.

- `references/mcp-ecosystem.md` — Complete catalog of official MCP tools (13) and community servers (figma-console-mcp with 57+ tools, cursor-talk-to-figma-mcp, etc.). Setup instructions, architecture diagrams, comparison table. Read when setting up Figma MCP or choosing which server to use.

- `references/design-workflow.md` — Step-by-step workflows for common design tasks: creating a page from scratch, editing existing designs, extracting design systems, building components. Read when performing specific design tasks.

- `references/gap-analysis.md` — Detailed comparison of Figma vs Paper vs Pencil capabilities, organized by category (read, write, edit, delete, workflow, design systems, AI features). Read when evaluating what's possible in Figma vs other tools or when planning what a custom plugin needs to implement.

## Common Patterns

### Extracting a node ID from a Figma URL
```
https://figma.com/design/ABC123/My-File?node-id=42-1337
→ nodeId = "42:1337"  (replace hyphen with colon)
```

### Setting up the official desktop MCP
```bash
# Claude Code
claude mcp add --transport http figma-desktop http://127.0.0.1:3845/mcp

# Requires: Figma Desktop app running, Dev Mode enabled (Shift+D)
```

### Setting up the official remote MCP
```bash
# Claude Code
claude mcp add --transport http figma https://mcp.figma.com/mcp
# Triggers OAuth flow for authentication
```

### The WebSocket + Plugin bridge pattern (community servers)
```
AI Agent ↔ MCP Server (local, Node.js) ↔ WebSocket ↔ Figma Plugin ↔ Plugin API (full read/write)
```
This is the universal architecture for write-capable Figma MCP servers. The plugin runs inside Figma Desktop and has full Plugin API access. The WebSocket bridge relays commands from the MCP server to the plugin.

---
name: figma-design
description: >
  Comprehensive Figma guide for AI agents working with design files, design-to-code,
  canvas automation, Plugin API scripts, MCP tools, variables, components, effects,
  CSS Grid, REST API, Code Connect, and design-system extraction. Use when reading,
  creating, editing, or validating Figma designs; building Figma plugins or scripts;
  setting up official/community Figma MCP servers; comparing Figma capabilities with
  Paper or Pencil; or handling Figma Design Agent, figma.com, Figma Desktop, Figma
  REST API, webhooks, Slides, Buzz, codegen, or design token workflows.
---

# Figma Design — AI Agent Skill

This skill equips you to work with Figma as a design tool at a level comparable to Paper and Pencil MCP integrations. The core challenge: Figma has multiple overlapping surfaces — built-in Design Agent, official MCP tools, `use_figma` write-to-canvas, REST API, and Plugin API — with different capability boundaries. This skill teaches the full landscape and the practical paths to full design capabilities.

## May 2026 Agent/Canvas Update

Figma's May 20, 2026 Design Agent launch adds an in-product agent on the Figma canvas. Do **not** treat this as a broad REST API expansion. For external coding agents, the relevant automation surface is **Figma MCP Server + `use_figma` + Plugin API**.

Use the layers this way:

- **Built-in Figma Design Agent**: product feature inside Figma Design for canvas prompting, edits, feedback, and design-system-aware work.
- **Figma MCP read tools**: `get_design_context`, `get_metadata`, `get_screenshot`, `get_variable_defs`, and design-system discovery for design-to-code, inspection, and validation.
- **`use_figma` / write-to-canvas**: Plugin API JavaScript in the file context for node creation/mutation, variables, styles, components, variants, exact file-context reads, and bulk edits. Load the `figma-use` guidance before writing scripts.
- **`generate_figma_design`**: prompt-to-canvas or code-to-canvas generation of editable Figma layers, followed by screenshot/metadata validation and targeted refinement.
- **Code Connect tools**: map Figma components to code components and improve component reuse during implementation.
- **REST API**: file/project/team metadata, exports, comments, versions, webhooks, and governance/admin workflows. Do not expect REST to expose full canvas authoring.

## Architecture Overview

Figma exposes three API surfaces with very different capabilities:

| API Surface | Read | Write (design nodes) | Requires Figma open? |
|---|---|---|---|
| **REST API** | Full file tree, all properties | No (only comments, variables*, dev resources) | No |
| **Plugin API** | Full file tree, all properties | Yes — full CRUD on all 37+ node types | Yes (runs inside Figma) |
| **Official MCP** | Design context, screenshots, metadata, variables, Code Connect, design-system search | Yes via `use_figma`, `generate_figma_design`, diagram generation, and Code Connect writes where available | Depends on tool: Desktop/local for file-context operations; Remote for OAuth-backed cloud workflows |

*Variables write requires Enterprise plan.

The **Plugin API** remains the path to full fine-grained design editing power. Official `use_figma` exposes that path to external agents through Figma MCP; community MCP servers often do the same through WebSocket + plugin bridges.

## What Tools Are Available?

### If you have official Figma MCP tools

You have Figma's official MCP surface. Tool availability varies between Desktop/local and Remote/OAuth deployments, but the practical groups are:

- `get_design_context` — primary design-to-code tool. Returns structured design data and usually code representation.
- `get_metadata` — sparse node map for large designs and targeted follow-up calls.
- `get_screenshot` — visual reference for parity checks.
- `get_variable_defs` — variables/styles used in the selection.
- `search_design_system` / library tools — discover reusable components, variables, styles, and libraries before recreating them.
- `use_figma` — write-to-canvas and exact file-context read path via Plugin API JavaScript. Load `figma-use` guidance first.
- `generate_figma_design` — generate editable design layers from prompts or code-oriented screen descriptions.
- `get_figjam` / `generate_diagram` — FigJam read and diagram creation workflows.
- Code Connect tools — read/add mappings between Figma nodes and code components.
- `create_design_system_rules` — generates project-specific design-to-code guidance.
- `whoami` — authenticated user identity/plan information where supported.

Use `get_design_context` as the primary inspection tool, then validate visually with `get_screenshot`. Use `use_figma` only when you need Plugin API execution in the file context.

### If you have `use_figma` or community write tools

You have Plugin API-backed read/write access. See `references/mcp-ecosystem.md` for details. Key write operations:
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
- `layoutSizingHorizontal/Vertical` — `'FIXED' | 'HUG' | 'FILL'` (preferred shorthand API)
  - `'HUG'` — only valid on auto-layout frames and text nodes (container shrinks to fit content)
  - `'FILL'` — only valid on direct children of auto-layout frames (child expands to fill parent)
  - `'FIXED'` — explicit size, always valid
- For artboards: use `layoutSizingVertical: 'HUG'` so height grows with content
- `layoutWrap: 'WRAP'` — only on HORIZONTAL frames
- `minWidth`/`maxWidth`/`minHeight`/`maxHeight` — only on auto-layout frames and their children, must apply AFTER `appendChild()`

See `references/plugin-api.md` section 7 for the complete auto layout reference including three API layers, old-to-new mapping tables, sizing patterns, and ordering requirements.

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

- `references/plugin-api.md` — Core Plugin API: 33 SceneNode types, creation methods, properties, 6 effect types, 5 paint types, auto layout + CSS Grid, variables, components, text, vectors, prototyping (with spring easing), styles, events. Updated for @figma/plugin-typings v1.123.0.

- `references/new-effects-paints.md` — **NEW** Deep reference for 2025 beta features: Glass effect, Progressive Blur, Noise (3 subtypes), Texture, Pattern Paint. Full TypeScript types, code examples, gotchas, effect limits per layer, timeline of API changes.

- `references/advanced-apis.md` — **NEW** 21 sections covering: CSS Grid layout, Annotations, Dev Resources, Measurements, Codegen plugins, Payments API, Team Library, Extended Variable Collections, Slides API, Buzz API, Draw APIs (TextPath, TransformGroup), JSX node creation, script execution & runtime, data storage, dynamic page loading, new text features, new prototyping features, new node types, deprecations & breaking changes (sync→async migration guide), REST API additions, webhooks.

- `references/mcp-ecosystem.md` — Complete catalog of official MCP tools (13) and community servers (figma-console-mcp with 57+ tools, cursor-talk-to-figma-mcp, etc.). Setup instructions, architecture diagrams, comparison table.

- `references/design-workflow.md` — Step-by-step workflows for common design tasks: creating a page from scratch, editing existing designs, extracting design systems, building components.

- `references/gap-analysis.md` — Figma vs Paper vs Pencil capability comparison across read, write, edit, delete, workflow, design systems, and AI features.

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

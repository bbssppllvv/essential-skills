# Gap Analysis: Figma vs Paper vs Pencil (AI Agent Capabilities)

## Table of Contents
1. [Capability Matrix](#1-capability-matrix)
2. [Read Capabilities](#2-read-capabilities)
3. [Write/Create Capabilities](#3-writecreate-capabilities)
4. [Edit Capabilities](#4-edit-capabilities)
5. [Delete Capabilities](#5-delete-capabilities)
6. [Design System & Variables](#6-design-system--variables)
7. [AI & Workflow](#7-ai--workflow)
8. [What Figma Needs to Match Paper/Pencil](#8-what-figma-needs)
9. [Architectural Comparison](#9-architectural-comparison)

---

## 1. Capability Matrix

| Capability | Paper MCP | Pencil MCP | Figma Official MCP | Figma + Community MCP |
|---|:---:|:---:|:---:|:---:|
| **Read design tree** | get_tree_summary, get_children, get_node_info | batch_get (patterns, IDs, depth) | get_metadata (XML) | Full Plugin API access |
| **Screenshot** | get_screenshot (1x/2x, JPEG/PNG) | get_screenshot | get_screenshot | Yes |
| **Get styles/CSS** | get_computed_styles, get_jsx | batch_get with properties | get_design_context (code) | Yes |
| **Get selection** | get_selection | get_editor_state | Selection-based (desktop) | Yes |
| **Get variables** | N/A | get_variables | get_variable_defs | Yes |
| **Create artboard/frame** | create_artboard | batch_design I() | No | Yes |
| **Create nodes** | write_html (HTML→nodes) | batch_design I() (per-node) | No* | Yes (all 37+ types) |
| **Edit text** | set_text_content | batch_design U() | No | Yes |
| **Edit styles** | update_styles (CSS) | batch_design U() | No | Yes |
| **Delete nodes** | delete_nodes | batch_design D() | No | Yes |
| **Duplicate** | duplicate_nodes (with ID map) | batch_design C() | No | Yes |
| **Move/reorder** | N/A | batch_design M() | No | Yes |
| **Replace nodes** | write_html replace mode | batch_design R() | No | Yes |
| **Rename** | rename_nodes | N/A | No | Yes |
| **Batch operations** | Yes (per-tool batching) | Yes (25 ops/call, atomic) | No | Depends on server |
| **Font info** | get_font_family_info | N/A | N/A | Yes (via Plugin API) |
| **Image generation** | N/A | G() AI/stock | No | Via Plugin API |
| **Design guidelines** | get_guide (1 topic) | get_guidelines (8 topics) | create_design_system_rules | N/A |
| **Style guides** | N/A | get_style_guide + tags | N/A | N/A |
| **Layout inspection** | N/A | snapshot_layout | N/A | N/A |
| **Property audit** | N/A | search_all_unique_properties | N/A | N/A |
| **Bulk replace** | N/A | replace_all_matching_properties | N/A | N/A |
| **Find empty space** | N/A | find_empty_space_on_canvas | N/A | N/A |
| **Working indicator** | finish_working_on_nodes | N/A | N/A | N/A |
| **Components** | Roadmap | Full (reusable + ref + slots) | Read only | Full via Plugin API |
| **Prototyping** | N/A | N/A | N/A | Full via Plugin API |

*`generate_figma_design` captures live UI as layers, but doesn't create individual nodes.

---

## 2. Read Capabilities

### Paper (11 tools)
Strong and granular. Separate tools for tree summary, children, node info, computed styles, JSX export, screenshots, font info, fill images, selection, basic info, and guides. Each tool does one thing well.

### Pencil (10 tools)
Consolidated but powerful. `batch_get` is a Swiss Army knife — search by patterns (name regex, type, reusable flag), batch-read by IDs, control traversal depth, resolve instances and variables. Plus dedicated tools for screenshots, variables, layout inspection, property auditing, style guides, guidelines, editor state, and canvas space finding.

### Figma Official (6 desktop / 9 remote tools)
Solid for code generation workflows. `get_design_context` is the star — returns framework-specific code + screenshot + metadata in one call. `get_metadata` provides structural XML. `get_variable_defs` extracts tokens. But no tree traversal, no computed styles, no per-node property inspection, no font lookup.

### Gap
Figma official MCP lacks:
- Fine-grained tree traversal (list children, inspect specific nodes)
- Computed CSS per-node (only via `get_design_context` which is higher-level)
- Font availability checking
- Layout/spatial inspection tools
- Design property auditing

---

## 3. Write/Create Capabilities

### Paper (2 tools)
- `create_artboard` — top-level frame with CSS styles
- `write_html` — the killer feature. Write HTML with inline CSS, Paper converts to design nodes. Agents already know HTML/CSS. Two modes: insert-children and replace.

### Pencil (batch_design I/C/G + open_document + set_variables)
- `I()` insert — create any node type with full property control
- `C()` copy — duplicate with auto-placement
- `G()` generate image — AI or stock photo fills
- `open_document` — create new or open existing .pen files
- `set_variables` — create/update design tokens

### Figma Official
- `generate_figma_design` (remote only) — captures running web UI as design layers. Not true node creation.
- `generate_diagram` — FigJam from Mermaid syntax. Not design nodes.
- `add_code_connect_map` / `send_code_connect_mappings` — metadata, not visual content.

### Gap
Figma official MCP has **no tool for creating design nodes**. This is the #1 gap. Paper solves it with HTML→nodes. Pencil solves it with a typed node insertion DSL. Community MCP servers solve it by bridging the Plugin API (full `figma.createFrame()`, `createText()`, etc.).

---

## 4. Edit Capabilities

### Paper (4 tools)
- `update_styles` — batch CSS property updates
- `set_text_content` — batch text changes
- `rename_nodes` — batch rename
- `duplicate_nodes` — deep clone with descendant ID map

### Pencil (batch_design U/R/M + replace_all_matching_properties)
- `U()` update — modify any node properties (except children)
- `R()` replace — swap a node entirely
- `M()` move — reorder in tree
- `replace_all_matching_properties` — bulk find-and-replace across design

### Figma Official
**None.** Zero edit capabilities for design nodes.

### Gap
Complete absence of edit tools in official MCP. Community servers fill this via Plugin API (`node.fills = ...`, `node.characters = ...`, etc.).

---

## 5. Delete Capabilities

### Paper
`delete_nodes` — batch delete by IDs.

### Pencil
`D()` in batch_design — single node delete, atomic with batch rollback.

### Figma Official
**None.**

### Gap
No delete capability in official MCP.

---

## 6. Design System & Variables

### Paper
Early stage. No variable system, no component model, no design tokens. Roadmap items: "components with slots", "themes and tokens".

### Pencil
Mature. 4 variable types, multi-axis theming, 379 components across 4 bundled design systems, component slots/overrides, `get_variables`/`set_variables` tools, design guidelines (8 topics), style guides.

### Figma Official
`get_variable_defs` reads variables. `create_design_system_rules` generates a rules document. REST API can read/write variables (Enterprise only). No component creation/editing via MCP.

### Figma + Community
Full access via Plugin API: create variables, collections, modes, bind to nodes, create components, component sets, instances, styles. The most powerful system of the three once Plugin API access is established.

### Gap
Figma has the most powerful underlying variable and component system (Plugin API level), but the official MCP barely exposes it. Community servers bridge this gap.

---

## 7. AI & Workflow

### Paper
- System prompt encodes specific aesthetic taste (minimalism, neutrals-first, one strong color moment)
- Mandatory screenshot review every 2-3 edits
- Required pre-generation design brief
- Incremental building (one visual group per write_html call)
- `finish_working_on_nodes` releases working indicators

### Pencil
- 8 topic-specific design guidelines (landing-page, mobile-app, web-app, etc.)
- Style guide system (230+ tags for creative direction)
- Anti-AI-Slop Rules embedded in guidelines
- Spawn up to 8-10 parallel sub-agents
- `find_empty_space_on_canvas` for intelligent placement
- `snapshot_layout` with problem detection

### Figma Official
- `create_design_system_rules` generates agent rules
- Code Connect for component-aware code generation
- No design quality enforcement
- No incremental building workflow
- No critique/review loop

### Gap
Figma lacks:
- Built-in design quality guidelines and taste encoding
- Incremental building workflow (write small, review often)
- Screenshot-based review loops mandated by the tool
- Style guide / creative direction system
- Layout problem detection
- Canvas space management

These are all things that would need to be encoded in a custom plugin's system prompt or companion instructions, not in the plugin code itself.

---

## 8. What Figma Needs to Match Paper/Pencil

### Already Possible (via Plugin API + community MCP)
1. Create any node type (37+ types vs Paper's 5, Pencil's 14)
2. Full style control (fills, strokes, effects, auto layout)
3. Text editing with character-level formatting
4. Variable/token management (more powerful than either competitor)
5. Component creation and management (most mature of the three)
6. Screenshot/export for review loops
7. Vector editing and boolean operations (beyond both competitors)
8. Prototyping (unique to Figma — neither Paper nor Pencil has this)

### Missing and Needs Building
1. **HTML→design conversion** (Paper's killer feature) — a plugin could parse HTML/CSS and create equivalent Figma nodes
2. **Batch operations with atomic rollback** (Pencil's batch_design) — a plugin could implement transaction semantics
3. **Design guidelines/taste system** (Pencil's 8 topics + Paper's prompt-as-taste) — needs custom agent instructions
4. **Style guide creative direction** (Pencil's tag system) — could be a companion dataset
5. **Layout problem detection** (Pencil's snapshot_layout) — a plugin could analyze spatial relationships
6. **Canvas space finding** (Pencil's find_empty_space) — a plugin could compute available space
7. **Property auditing** (Pencil's search_all_unique_properties) — a plugin could traverse and collect
8. **Bulk property replacement** (Pencil's replace_all_matching_properties) — a plugin could find-and-replace
9. **Working/progress indicators** (Paper's finish_working_on_nodes) — could use plugin UI
10. **Incremental rendering feedback** — a plugin could provide real-time progress to agents

---

## 9. Architectural Comparison

| Dimension | Paper | Pencil | Figma |
|---|---|---|---|
| **Canvas model** | HTML/CSS-native | Skia/WASM vectors | Proprietary (C++/WASM) |
| **Write language** | HTML (agents know it) | Custom DSL (compact) | Plugin API (JavaScript) |
| **MCP transport** | Hono HTTP in-process | Go binary STDIO | HTTP (official) or WebSocket (community) |
| **Data model** | Cloud-first, server-synced | Local JSON (.pen) | Cloud-first, proprietary format |
| **AI provider** | Any MCP-capable agent | Built-in Claude + Codex | Any MCP-capable agent |
| **Collaboration** | Real-time multiplayer | Git-based | Real-time multiplayer (industry-leading) |
| **Ecosystem** | Small, new | Growing | Massive (20M+ users, thousands of plugins) |
| **Node types** | 5 confirmed | 14 | 37+ |
| **Component system** | Roadmap | Mature | Industry-standard (most mature) |

### Key Insight
Figma has the most powerful underlying platform but the weakest AI/agent integration. Paper and Pencil were built AI-first — their entire architecture serves the agent workflow. Figma added AI capabilities on top of an existing product. A well-designed Figma plugin + custom MCP server could theoretically surpass both Paper and Pencil, because:

1. Figma's Plugin API is more powerful than either competitor's write surface
2. Figma's component/variable system is the most mature
3. Figma's ecosystem (fonts, plugins, community files) is vastly larger
4. Figma's collaboration is industry-leading

The bottleneck is purely in the **bridge quality** — how well the MCP server translates AI intent into Plugin API calls, and how well the system prompt encodes design taste and quality standards.

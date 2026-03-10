# Figma MCP Ecosystem — Official + Community Servers

## Table of Contents
1. [Official Figma MCP Server](#1-official-figma-mcp-server)
2. [Community Write-Capable Servers](#2-community-write-capable-servers)
3. [Server Comparison Matrix](#3-server-comparison-matrix)
4. [Setup Instructions](#4-setup-instructions)
5. [Architecture: WebSocket + Plugin Bridge](#5-architecture)
6. [Rate Limits](#6-rate-limits)

---

## 1. Official Figma MCP Server

Two variants — Remote (cloud-hosted) and Desktop (local).

### All 13 Official Tools

| # | Tool | Read/Write | Remote | Desktop | Description |
|---|---|---|---|---|---|
| 1 | `get_design_context` | Read | Yes | Yes | Returns code (React+Tailwind default) + screenshot + metadata |
| 2 | `get_metadata` | Read | Yes | Yes | Sparse XML tree — IDs, types, names, positions, sizes |
| 3 | `get_screenshot` | Read | Yes | Yes | Screenshot of a node or selection |
| 4 | `get_variable_defs` | Read | Yes | Yes | Design token values (colors, spacing, typography) |
| 5 | `get_code_connect_map` | Read | Yes | No | Figma node → code component mappings |
| 6 | `get_code_connect_suggestions` | Read | Yes | No | Auto-detected component mappings |
| 7 | `get_figjam` | Read | Yes | Yes | FigJam board content in XML |
| 8 | `create_design_system_rules` | Read* | Yes | Yes | Generates design system ruleset |
| 9 | `generate_figma_design` | **Write** | **Yes** | No | Sends live UI screenshots as design layers |
| 10 | `generate_diagram` | **Write** | Yes | No | Creates FigJam diagram from Mermaid syntax |
| 11 | `add_code_connect_map` | **Write** | Yes | No | Creates node → code mappings |
| 12 | `send_code_connect_mappings` | **Write** | Yes | No | Confirms detected mappings |
| 13 | `whoami` | Read | **Yes** | No | User identity and plan info |

*`create_design_system_rules` creates a rules file, not design nodes.

### Key Limitation
`generate_figma_design` is the only tool that creates visual content on canvas, but it works by **capturing screenshots of running web UIs** and converting them to Figma layers — it does NOT create designs from scratch or edit existing ones. There is no official MCP tool for creating/editing individual design nodes.

### Desktop MCP Parameters

All desktop tools accept:
- `nodeId` (optional) — target node ID (`"123:456"` format). Defaults to current selection.
- `clientLanguages` (optional) — for logging (e.g., `"typescript"`)
- `clientFrameworks` (optional) — for logging (e.g., `"react"`)

`get_design_context` also accepts:
- `artifactType` — WEB_PAGE_OR_APP_SCREEN, COMPONENT_WITHIN_A_WEB_PAGE_OR_APP_SCREEN, REUSABLE_COMPONENT, DESIGN_SYSTEM
- `taskType` — CREATE_ARTIFACT, CHANGE_ARTIFACT, DELETE_ARTIFACT
- `forceCode` — force code output even for large designs

---

## 2. Community Write-Capable Servers

All use the same architecture: **MCP Server ↔ WebSocket ↔ Figma Plugin ↔ Plugin API (full read/write)**.

### figma-console-mcp (Southleft) — Most Comprehensive
- **57+ tools**, production-ready (v1.11.2)
- **Repo**: github.com/southleft/figma-console-mcp
- **Docs**: figma-console-mcp.southleft.com
- Two modes: Local (57+ tools, full read/write) and Remote (22 tools, read-only via Cloudflare Workers)
- WebSocket on ports 9223-9232 with auto multi-instance fallback
- 11 dedicated variable management tools (batch operations, 100 tokens/call)
- Design token export: CSS custom properties, Tailwind config, Sass variables
- Real-time console log capture from Figma plugins

### figma-mcp-write-server (oO)
- **24 tools** organized in categories
- **Repo**: github.com/oO/figma-mcp-write-server
- Categories:
  - Core Design: nodes, text, fills, strokes, effects
  - Layout: auto_layout, constraints, alignment, hierarchy
  - Design System: styles, components, instances, variables, fonts
  - Advanced: boolean_operations, vectors
  - Developer: dev_resources, annotations, measurements, exports
- Requires Node.js v22.x

### cursor-talk-to-figma-mcp (Grab)
- **~15+ tools**
- **Repo**: github.com/grab/cursor-talk-to-figma-mcp
- Query documents, inspect nodes, create elements, adjust styling, export assets
- Originally built for Cursor, works with Claude Code

### claude-talk-to-figma-mcp (arinspunk)
- **~15+ tools**
- **Repo**: github.com/arinspunk/claude-talk-to-figma-mcp
- Fork/variant optimized for Claude Desktop and Claude Code

### Figma-MCP-Write-Bridge (firasmj)
- **Repo**: github.com/firasmj/Figma-MCP-Write-Bridge
- Lightweight plugin bridge for write/manipulation

### mcp-figma (thirdstrandstudio)
- **Repo**: github.com/thirdstrandstudio/mcp-figma
- Exposes full REST API through MCP (no desktop app required, but read-only for design data)

### Alternative: Browser-Based Plugin API Access
- No MCP needed — give Claude a browser MCP, navigate to Figma, execute Plugin API code directly
- Documented at cianfrani.dev/posts/a-better-figma-mcp/
- Advantage: unlimited access to full Plugin API
- Disadvantage: requires browser control, less structured

---

## 3. Server Comparison Matrix

| Server | Write? | Tools | Needs Figma Open? | Free? | Best For |
|---|---|---|---|---|---|
| Official Remote | Limited* | 13 | No | Paid plans | Design-to-code, reading |
| Official Desktop | No | 6 | Yes | Paid plans | Quick inspection |
| figma-console-mcp | **Full** | 57+ | Yes | Yes | Full AI design workflow |
| figma-mcp-write-server | **Full** | 24 | Yes | Yes | Structured design ops |
| cursor-talk-to-figma-mcp | **Full** | ~15 | Yes | Yes | Basic create/edit |
| mcp-figma | Read-only | REST API | No | Yes | REST API access |

*Limited write: `generate_figma_design` captures live UI, doesn't create from scratch.

---

## 4. Setup Instructions

### Official Desktop MCP
```bash
# Claude Code
claude mcp add --transport http figma-desktop http://127.0.0.1:3845/mcp

# Requirements:
# 1. Figma Desktop app (latest version)
# 2. Dev Mode enabled (Shift+D)
# 3. Dev or Full seat on paid plan
# 4. MCP toggle enabled in inspect panel
```

### Official Remote MCP
```bash
# Claude Code
claude mcp add --transport http figma https://mcp.figma.com/mcp
# Triggers OAuth flow in browser

# VS Code settings.json
{
  "mcpServers": {
    "figma": { "url": "https://mcp.figma.com/mcp", "type": "http" }
  }
}
```

### figma-console-mcp (recommended for write access)
```bash
# 1. Install the MCP server
npm install -g figma-console-mcp

# 2. Install the Desktop Bridge plugin in Figma
# (available from Figma Community or the repo's releases)

# 3. Configure Claude Code
claude mcp add figma-console -- npx figma-console-mcp

# 4. Start Figma Desktop and run the Desktop Bridge plugin
# The plugin connects via WebSocket on ports 9223-9232
```

### cursor-talk-to-figma-mcp
```bash
# 1. Clone the repo
git clone https://github.com/grab/cursor-talk-to-figma-mcp
cd cursor-talk-to-figma-mcp

# 2. Install dependencies
npm install

# 3. Build and start MCP server
npm run build && npm start

# 4. Install the companion Figma plugin (from the repo's figma-plugin/ directory)
# 5. Run the plugin in Figma — it connects via WebSocket
```

---

## 5. Architecture

### Why WebSocket + Plugin Bridge?

Figma's REST API is **read-only for design data**. The only API with full write access is the **Plugin API**, which runs inside Figma's sandboxed environment. Community servers work around this by:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  AI Agent    │────▶│  MCP Server  │────▶│  WebSocket   │────▶│ Figma Plugin │
│  (Claude)    │◀────│  (Node.js)   │◀────│  Bridge      │◀────│ (Plugin API) │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                      localhost:PORT       ws://PORT+1          Inside Figma
```

1. AI agent sends MCP tool call (e.g., "create rectangle")
2. MCP server translates to Plugin API command
3. WebSocket bridge relays to Figma plugin
4. Plugin executes via Plugin API (full read/write access)
5. Result propagates back

### Requirements
- Figma Desktop app must be running
- The companion plugin must be active
- WebSocket connection must be established

### Gotchas
- Plugin can crash or be closed → connection breaks
- Only one plugin at a time in Figma → can't use other plugins simultaneously
- Port conflicts when running multiple Figma instances
- Font loading is async → text operations need proper sequencing

---

## 6. Rate Limits

### Official MCP (Daily Limits)

| Plan | Daily Limit |
|---|---|
| Enterprise | 600/day |
| Organization/Pro (Full or Dev seat) | 200/day |
| Starter / View / Collab seats | 6/month |

### Per-Minute Limits

| Seat Type | Starter | Pro | Org | Enterprise |
|---|---|---|---|---|
| View/Collab | 6/month | 6/month | 6/month | 6/month |
| Dev/Full | 10/min | 15/min | 20/min | — |

### Rate-Limit-Exempt Tools
- `add_code_connect_map`
- `generate_figma_design`
- `whoami`

### Community Servers
No rate limits — limited only by Plugin API execution speed and WebSocket throughput. Typical latency: 50-200ms per operation.

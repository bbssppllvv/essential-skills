# essential-skills

A collection of Claude Code skills — modular reference packages for AI-assisted development.

## Available Skills

| Skill | Description |
|-------|-------------|
| [agent-engineering](agent-engineering/) | Production-grade AI agents, harnesses, tool loops, evals, permissions, multi-agent patterns, and runbooks. |
| [agent-folder-init](agent-folder-init/) | Add or repair `.agents/` project context, agent docs, session tracking, task management, and coding standards. |
| [openrouter](openrouter/) | Complete OpenRouter API documentation (178 pages). Routing, models, SDKs, integrations. |
| [figma-design](figma-design/) | Figma MCP, Plugin API, REST API, design-to-code, canvas automation, and AI design workflows. |
| [fluidaudio](fluidaudio/) | FluidAudio Swift SDK for on-device audio AI on Apple platforms. ASR (Parakeet TDT), diarization, VAD, TTS via CoreML/ANE. |
| [polar-integration](polar-integration/) | Polar payments, subscriptions, checkout, webhooks, and customer portal. SDK integration for @polar-sh. |
| [sayless](sayless/) | Copywriting methodology — write copy people read, trust, remember, and act on. |
| [soniox](soniox/) | Soniox STT/TTS APIs — real-time WebSocket streaming, async transcription, SDKs, and voice-agent patterns. |
| [product-design](product-design/) | Product design best practices — typography, color, spacing, motion, icons, accessibility, anti-AI-slop. |
| [vercel-ai](vercel-ai/) | Vercel AI ecosystem — AI SDK v6, AI Elements (48 components), Chat SDK, Workflow DevKit, AI Gateway, Streamdown, Data Stream Protocol. |
| [crucible](crucible/) | Independent multi-perspective subagent review panels for high-risk plans, PRs, decisions, and debugging. |

## Usage

Each folder is a self-contained skill with a `SKILL.md` and any bundled `references/`, `scripts/`, or `assets/` it needs.

Install a skill into Claude Code:

```bash
claude skill install /path/to/skill-folder
```

Or reference files directly as context.

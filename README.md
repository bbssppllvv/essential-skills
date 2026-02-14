# essential-skills

A collection of Claude Code skills — modular reference packages for AI-assisted development.

## Available Skills

| Skill | Description |
|-------|-------------|
| [openrouter](openrouter/) | Complete OpenRouter API documentation (178 pages). Routing, models, SDKs, integrations. |
| [fluidaudio](fluidaudio/) | FluidAudio Swift SDK for on-device audio AI on Apple platforms. ASR (Parakeet TDT), diarization, VAD, TTS via CoreML/ANE. |
| [polar-integration](polar-integration/) | Polar payments, subscriptions, checkout, webhooks, and customer portal. SDK integration for @polar-sh. |
| [sayless](sayless/) | Copywriting methodology — write copy people read, trust, remember, and act on. |
| [soniox](soniox/) | Soniox speech-to-text API — real-time WebSocket streaming, async REST transcription, Python/Node/Web SDKs. |

## Usage

Each folder is a self-contained skill with a `SKILL.md` and `references/` directory.

Install a skill into Claude Code:

```bash
claude skill install /path/to/skill-folder
```

Or reference files directly as context.

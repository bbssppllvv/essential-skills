# essential-skills

A collection of Claude Code skills — modular reference packages for AI-assisted development.

## Available Skills

| Skill | Description |
|-------|-------------|
| [openrouter](openrouter/) | Complete OpenRouter API documentation (178 pages). Routing, models, SDKs, integrations. |
| [fluidaudio](fluidaudio/) | FluidAudio Swift SDK for on-device audio AI on Apple platforms. ASR (Parakeet TDT), diarization, VAD, TTS via CoreML/ANE. |

## Usage

Each folder is a self-contained skill with a `SKILL.md` and `references/` directory.

Install a skill into Claude Code:

```bash
claude skill install /path/to/skill-folder
```

Or reference files directly as context.

# Skills Loader - OpenRouter SDK Documentation

## Overview

This guide demonstrates building encapsulated tools that inject domain-specific context into conversations. When loaded, skills automatically enrich subsequent turns with specialized instructions, similar to Claude Code functionality.

## Setup Requirements

Install dependencies:
```bash
pnpm add @openrouter/sdk zod
```

Create skill directories:
```bash
mkdir -p ~/.claude/skills/pdf-processing
mkdir -p ~/.claude/skills/data-analysis
mkdir -p ~/.claude/skills/code-review
```

## Core Implementation

The basic skills tool uses `nextTurnParams` to inject skill context. Key features include:

- **Skill Discovery**: Lists available skills from the filesystem
- **Context Injection**: Appends skill instructions to conversation
- **Idempotency**: Prevents duplicate skill loading through markers
- **Error Handling**: Gracefully manages missing skills

## Skill File Format

Skills are stored as markdown files (e.g., `SKILL.md`) containing:
- Tool descriptions and capabilities
- Best practices and guidelines
- Output format specifications
- Error handling strategies

## Advanced Patterns

**Multi-Skill Loading**: Load multiple skills simultaneously for complex tasks

**Configurable Skills**: Accept options for verbosity, output format, and operational modes

**Skill Discovery Tool**: Lists available skills with descriptions and configuration status

## Key Design Principles

1. **Idempotency**: Check for existing markers before injecting duplicate content
2. **Graceful Fallbacks**: Handle missing skills without breaking conversation flow
3. **Context Preservation**: Always append to existing input rather than replacing it
4. **Clear Markers**: Use unique identifiers to tag injected content for reliable detection

## Integration Example

Combine multiple skill tools in a single model call to enable progressive skill loading and discovery throughout the conversation flow.

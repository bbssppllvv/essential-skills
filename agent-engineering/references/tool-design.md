# Tool Design — Deep Dive

Evidence basis:
- [Anthropic: Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic: Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- [Anthropic tool use docs](https://platform.claude.com/docs/en/docs/agents-and-tools/tool-use/overview)
- [Vercel AI SDK tool-calling docs](https://ai-sdk.dev/docs/foundations/tools)
- [OpenAI Agents SDK guardrails docs](https://openai.github.io/openai-agents-js/guides/guardrails/)

Treat tool descriptions as model-facing API documentation. Treat permissions, secrets, and side effects as harness policy, not prompt policy.

## Table of Contents
1. [Tool Definition Anatomy](#tool-definition-anatomy)
2. [Writing Great Descriptions](#writing-great-descriptions)
3. [Parameter Design](#parameter-design)
4. [Result Design](#result-design)
5. [Tool Registry Pattern](#tool-registry-pattern)
6. [Common Tool Patterns](#common-tool-patterns)
7. [Anti-Patterns](#anti-patterns)

---

## Tool Definition Anatomy

Every LLM API expects tools defined as JSON Schema. The three parts that matter:

```json
{
  "name": "edit_file",
  "description": "Replace a specific string in a file with new content. ...",
  "input_schema": {
    "type": "object",
    "properties": {
      "file_path": {
        "type": "string",
        "description": "Absolute path to the file"
      },
      "old_string": {
        "type": "string", 
        "description": "Exact text to find and replace. Must match uniquely."
      },
      "new_string": {
        "type": "string",
        "description": "Text to replace old_string with"
      }
    },
    "required": ["file_path", "old_string", "new_string"]
  }
}
```

The LLM reads **name**, **description**, and **parameter descriptions** to decide when and how to use the tool. These are your documentation — treat them seriously.

---

## Production Contract Fields

For production tools, the model-facing schema is only part of the contract. The harness should also store metadata the permission, execution, and observability layers can enforce:

```text
purpose
input_schema
output_schema
risk_class
side_effect_class
resource_scope
permission_policy
timeout
result_size_limit
retry_policy
audit_policy
error_format
```

Do not depend on the model to infer this from prose. A `send_email` tool and a `draft_email` tool may look similar to the model, but the harness must treat them as different side-effect classes.

Useful risk classes:

```text
read_only
compute_only
draft_only
write_local
write_internal
write_external
external_communication
financial
identity_access
security_sensitive
process_execution
network_open_world
destructive
privileged_admin
```

Every side-effecting tool should have a policy decision before execution: allow, deny, ask user, require approval, require stronger auth, run in sandbox, or downgrade to draft-only.

---

## Writing Great Descriptions

The tool description is the most important thing you write. The LLM decides whether to use a tool based almost entirely on the description.

### Good descriptions:
- Say **what the tool does** in the first sentence
- Explain **when to use it** vs. alternatives
- Document **edge cases and constraints**
- Give **examples** of typical inputs

### Example: Read File (weak)
```
"Reads a file from disk."
```

### Example: Read File (strong)
```
"Read the contents of a file at the given path. Returns the full file content as text.
Use this to inspect source code, configuration files, or any text file before editing.
The path must be absolute. If the file doesn't exist, returns an error message.
For very large files (>10K lines), consider reading specific line ranges instead."
```

The strong version tells the LLM:
- What the tool returns (full content as text)
- When to use it (before editing, for inspection)
- Constraints (absolute paths)
- What happens on error (returns error message)
- Performance consideration (large files)

### Parameter descriptions matter too

```json
"old_string": {
  "type": "string",
  "description": "The exact text to find in the file. Must appear exactly once — if it appears multiple times, the edit will fail. Include enough surrounding context (a few lines) to make the match unique."
}
```

This prevents the most common edit_file failure: non-unique matches.

---

## Parameter Design

### Keep parameters minimal
Every parameter the LLM has to fill is a chance for error. If a value can be derived, derive it.

**Bad**: `edit_file(file_path, line_number, old_text, new_text, encoding, create_if_missing)`
**Good**: `edit_file(file_path, old_string, new_string)` — encoding is auto-detected, and file creation is a separate `write_file` or `create_file` tool so the edit semantics stay unambiguous.

### Use enums for constrained choices
```json
"urgency": {
  "type": "string",
  "enum": ["low", "normal", "high", "critical"],
  "description": "How urgent this task is"
}
```

Enums prevent the LLM from inventing values like "medium-high" or "URGENT!!!".

### Make optional parameters truly optional
Don't require parameters that have sensible defaults. The LLM shouldn't have to guess what `encoding="utf-8"` means every time.

---

## Result Design

### Return context, not just status

**Bad result:**
```json
{"status": "success"}
```

**Good result:**
```json
{
  "status": "success",
  "file": "/src/auth.py",
  "lines_changed": 3,
  "preview": "Line 42: def authenticate(user, token):  # ← edited"
}
```

The good result tells the LLM what happened, confirms which file changed, and shows a preview so it can verify correctness.

### Error results should be actionable

**Bad error:**
```json
{"error": "Operation failed"}
```

**Good error:**
```json
{"error": "old_string not found in /src/auth.py. The file contains 156 lines. Did you mean one of these similar strings?\n- Line 42: 'def authenticate(user, token):'\n- Line 89: 'def authenticate_session(session):'\nTip: re-read the file to get the current content."}
```

The good error tells the LLM exactly what went wrong and suggests how to fix it. This dramatically reduces stuck loops.

### Truncate large results

If a tool would return 50KB of text, truncate it:
```python
def read_file(path):
    content = open(path).read()
    if len(content) > MAX_RESULT_SIZE:
        return (
            f"File is {len(content)} chars ({content.count(chr(10))} lines). "
            f"Showing first {MAX_LINES} lines:\n\n"
            + "\n".join(content.split("\n")[:MAX_LINES])
            + f"\n\n... truncated. Use offset/limit to read specific ranges."
        )
    return content
```

---

## Tool Registry Pattern

A clean pattern for managing tools:

```typescript
import { readFileSync, writeFileSync } from "fs";

interface ToolDef {
  name: string;
  description: string;
  input_schema: Record<string, any>;
  execute: (args: Record<string, any>) => string;
}

const TOOL_REGISTRY = new Map<string, ToolDef>();

function registerTool(def: ToolDef) {
  if (TOOL_REGISTRY.has(def.name)) {
    throw new Error(`Duplicate tool name: ${def.name}`);
  }
  TOOL_REGISTRY.set(def.name, def);
}

registerTool({
  name: "read_file",
  description: "Read the contents of a file at the given absolute path.",
  input_schema: {
    type: "object",
    properties: { file_path: { type: "string", description: "Absolute path to the file" } },
    required: ["file_path"],
  },
  execute: ({ file_path }) => readFileSync(file_path, "utf-8"),
});

registerTool({
  name: "edit_file",
  description: "Replace old_string with new_string in a file. old_string must appear exactly once.",
  input_schema: {
    type: "object",
    properties: {
      file_path: { type: "string" },
      old_string: { type: "string", description: "Exact text to find. Must be unique in the file." },
      new_string: { type: "string" },
    },
    required: ["file_path", "old_string", "new_string"],
  },
  execute: ({ file_path, old_string, new_string }) => {
    const content = readFileSync(file_path, "utf-8");
    if (!content.includes(old_string)) return `Error: old_string not found in ${file_path}`;
    const count = content.split(old_string).length - 1;
    if (count > 1) return `Error: old_string appears ${count} times. Make it unique.`;
    writeFileSync(file_path, content.replace(old_string, new_string));
    return `Successfully edited ${file_path}`;
  },
});

// Convert to LLM-ready format (strip execute functions)
function getToolDefinitions() {
  return [...TOOL_REGISTRY.values()].map(({ name, description, input_schema }) => ({
    name, description, input_schema,
  }));
}

function executeTool(name: string, args: Record<string, any>): string {
  const tool = TOOL_REGISTRY.get(name);
  if (!tool) return `Error: unknown tool '${name}'`;
  try {
    return tool.execute(args);
  } catch (e) {
    return `Error executing ${name}: ${e instanceof Error ? e.message : String(e)}`;
  }
}
```

Registry rules:
- Reject duplicate names. Tool shadowing makes traces misleading and can route a safe-looking name to a dangerous implementation.
- Namespace remote or connector tools by source, for example `github.create_issue` vs. `linear.create_issue`, unless the harness has a deliberate alias layer.
- Keep a separate internal contract for risk class, side-effect class, permissions, timeout, output caps, and audit policy. Do not expose every internal field to the model.
- Version tool bundles and record the active bundle hash in every trace.

---

## Common Tool Patterns

### File Operations
| Tool | Purpose | Key detail |
|------|---------|------------|
| read_file | Read file contents | Support line ranges for large files |
| edit_file | Find-and-replace | Require unique match, not line numbers |
| write_file | Create/overwrite file | Use sparingly — edit is safer |
| glob | Find files by pattern | Return paths sorted by relevance |
| grep | Search file contents | Return file + line number + context |

### Shell Execution
| Tool | Purpose | Key detail |
|------|---------|------------|
| bash | Run shell command | Timeout, cwd, stderr, sandbox, command policy |
| run_tests | Execute test suite | Parse output for pass/fail |

### Web/Network
| Tool | Purpose | Key detail |
|------|---------|------------|
| web_fetch | Get URL content | Convert HTML to markdown |
| web_search | Search the internet | Return top N results with snippets |

### Agent-Specific
| Tool | Purpose | Key detail |
|------|---------|------------|
| think | Internal reasoning | Returns "OK" — value is in articulation |
| delegate | Spawn sub-agent | New conversation, isolated context |
| ask_user | Request clarification | Breaks inner loop, returns to user |

---

## The "Fewest Tools" Evidence

Vercel built an agent (d0) with 15+ specialized tools — schema lookup, validation, recovery, etc. — and achieved 80% success with high fragility. They replaced everything with a single bash execution tool in a sandbox, giving the model raw access to well-structured files via `grep`, `cat`, `find`, `ls`.

Results:

| Metric | Before (15+ tools) | After (1 bash tool) |
|--------|-------------------|---------------------|
| Success rate | 80% | 100% |
| Execution time | 274.8s | 77.4s (3.5x faster) |
| Token usage | ~102K | ~61K (37% less) |
| Steps required | ~12 | ~7 (42% fewer) |

The key insight: **"We were constraining reasoning because we didn't trust the model to reason."** With a capable model, hand-coded filters and pre-processed context become counterproductive — they prevent the model from applying its own judgment.

Two preconditions for this to work:
1. The model must be capable enough to reason about raw file access
2. The files/documentation must be well-structured so raw access is meaningful

This doesn't mean "always use one tool." It means: start by asking whether each specialized tool is solving a model reasoning problem or a harness engineering problem. If it's the former — remove the constraint and let the model reason.

Source: [Vercel: We removed 80% of our agent's tools (Dec 2025)](https://vercel.com/blog/we-removed-80-percent-of-our-agents-tools)

---

## Anti-Patterns

### God Tool
**Bad**: A single `do_anything(action, target, options)` tool that tries to handle everything.
**Why it's bad**: The LLM has to figure out the right action/target/options combination from a massive space. Separate tools with clear names guide the LLM to the right action.

### Stateful Tools
**Bad**: Tools that depend on hidden state (`select_file("auth.py")` then `edit_selected(old, new)`).
**Why it's bad**: If the LLM loses track of state, everything breaks. Each tool call should be self-contained — pass the file path every time.

Some harness state is fine (session IDs, auth handles, current sandbox), but tool calls should not require the model to remember invisible mutable state to be correct.

### Chatty Tools
**Bad**: Tools that return conversational text like "I've successfully updated the file for you!"
**Why it's bad**: The LLM parses tool results as data. Return structured, factual information. The LLM will compose the human-friendly message itself.

### Missing Error Information
**Bad**: `return "Error"` or catching exceptions silently.
**Why it's bad**: The LLM needs error details to recover. A bare "Error" sends it into a blind retry loop.

### Overly Fine-Grained Tools
**Bad**: `open_file()`, `seek_to_line()`, `read_line()`, `close_file()`
**Why it's bad**: Too many steps for a simple operation. The LLM has to make 4 tool calls to read one file. Combine into `read_file(path, start_line, end_line)`.

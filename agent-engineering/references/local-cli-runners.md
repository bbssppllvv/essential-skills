# Local CLI Agent Runners

How to build a low-level local runner for a Claude Code / Codex / Aider / mini-SWE-agent style agent on a developer machine, especially macOS.

This page is about the harness layer below "agent framework" abstractions: process execution, patch application, workspace trust, sandboxing, approvals, terminal UX, event logs, and replay. Use it when you need to design or implement a local coding/automation runner rather than only use an SDK.

Evidence basis:
- [OpenAI: Unrolling the Codex agent loop](https://openai.com/index/unrolling-the-codex-agent-loop/)
- [OpenAI: Unlocking the Codex harness / App Server](https://openai.com/index/unlocking-the-codex-harness/)
- [OpenAI Codex CLI docs](https://developers.openai.com/codex/cli)
- [OpenAI Responses API local shell docs](https://developers.openai.com/api/docs/guides/tools-local-shell)
- [OpenAI Codex sandboxing and approvals docs](https://developers.openai.com/codex/agent-approvals-security)
- [OpenAI Codex repository](https://github.com/openai/codex)
- [OpenAI Codex app-server protocol README](https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md)
- [OpenAI Codex protocol v1 notes](https://github.com/openai/codex/blob/main/codex-rs/docs/protocol_v1.md)
- [OpenAI GPT-5.2-Codex system card addendum](https://cdn.openai.com/pdf/ac7c37ae-7f4c-4442-b741-2eabdeaf77e0/oai_5_2_Codex.pdf)
- [Claude Code permissions docs](https://code.claude.com/docs/en/permissions)
- [Claude Code settings docs](https://code.claude.com/docs/en/settings)
- [Claude Code hooks docs](https://code.claude.com/docs/en/hooks)
- [Claude Agent SDK permissions docs](https://code.claude.com/docs/en/agent-sdk/permissions)
- [mini-SWE-agent docs and repository](https://mini-swe-agent.com/latest/usage/mini/)
- [SWE-agent paper](https://arxiv.org/abs/2405.15793)
- [SWE-agent ACI notes](https://github.com/SWE-agent/SWE-agent/blob/main/docs/background/aci.md)
- [Aider repository map docs](https://aider.chat/docs/repomap.html)
- [Aider edit format docs](https://aider.chat/docs/more/edit-formats.html)
- [Apple App Sandbox docs](https://developer.apple.com/documentation/security/app_sandbox)
- [Apple App Sandbox entitlement reference](https://developer.apple.com/library/archive/documentation/Miscellaneous/Reference/EntitlementKeyReference/Chapters/EnablingAppSandbox.html)
- [Apple: Embedding a command-line tool in a sandboxed app](https://developer.apple.com/documentation/Xcode/embedding-a-helper-tool-in-a-sandboxed-app)
- [Microsoft: Defend against indirect prompt injection attacks](https://learn.microsoft.com/en-us/security/zero-trust/sfi/defend-indirect-prompt-injection)
- [OWASP Top 10 for Agentic Applications](https://genai.owasp.org/2025/12/09/owasp-genai-security-project-releases-top-10-risks-and-mitigations-for-agentic-ai-security/)
- [Terminal Is All You Need](https://arxiv.org/abs/2603.10664)
- [Building Effective AI Coding Agents for the Terminal](https://arxiv.org/abs/2603.05344)

Treat exact CLI flags, config keys, SDK names, and model IDs as volatile. Verify same-day before implementation.

Use the source hierarchy from `source-map.md`: official docs and official repos define facts; papers and preprints provide research evidence; third-party analysis is hypothesis material only. Do not base runner security controls on Reddit, wiki pages, benchmark marketing, or unaudited blog summaries.

---

## Source-Backed Insights

These are the highest-value conclusions from the current source set. Each is either directly stated by a source or explicitly labeled as an implementation implication.

| Insight | Strongest sources | Implementation consequence |
|---------|-------------------|----------------------------|
| The UI should be a client of a local runner, not the runner itself. | Codex App Server docs and Codex protocol notes describe UI/client messages flowing to a local core over JSON-RPC or queues. | Build the agent core as a local service/library with an event protocol; let CLI, TUI, SwiftUI, IDE panels, and tests all consume the same events. |
| A "turn" is an event stream containing messages, tool calls, command output, approvals, file edits, and completion metadata. | Codex App Server lifecycle and protocol docs. | Do not expose only final text. Persist and render started/completed item events, approval requests, command chunks, patch events, errors, and token/cost usage. |
| Local shell is not "the model running commands"; the API/model emits command instructions and the runner decides whether and how to execute them. | OpenAI local shell docs. | Treat the model as untrusted input. Authorization, sandboxing, env filtering, timeouts, and redaction live in the harness. |
| Terminal-native agents work because the terminal is a text-native ACI: commands, code, logs, diffs, and tool parameters share the model's medium. | `Terminal Is All You Need`; SWE-agent ACI paper. | Prefer textual, inspectable primitives before GUI perception. If building a GUI, make it render the same command/diff/event stream rather than hiding it. |
| Tool/interface design measurably matters. SWE-agent reports that custom ACI design improves repository navigation, editing, testing, and empty-output handling. | SWE-agent paper and ACI notes. | Invest in observation formatting, file viewers, search result shape, patch grammar, and error messages as product surface, not plumbing. |
| Approvals and sandboxing are separate mechanisms. | Codex sandbox/approval docs; Claude permissions docs. | An approval dialog is not a security boundary. Enforce path, command, network, and secret rules before launch and again at OS/process boundaries. |
| Read/Edit deny rules do not necessarily constrain shell subprocesses. | Claude Code permissions docs explicitly distinguish built-in file tools from Bash subprocesses. | If Bash or subprocess execution exists, use OS-level sandboxing and command/path policy. Do not rely on file-tool deny rules alone. |
| Permission rule matching around shell commands is fragile. | Claude Code permissions docs discuss wrappers, `find -exec`, `curl`, globs, and compound commands. | Parse commands where possible, maintain conservative unsafe patterns, prefer exact allow rules for executable wrappers, and treat network filtering in Bash as hard. |
| Workspace trust is a hard boundary for project-local config. | Codex config docs load project `.codex/config.toml` only after trust; Claude settings distinguish user/project/local/managed scopes. | Never let a cloned repo enable hooks, MCP servers, skills, or looser permissions until the user trusts the workspace. Show config source precedence in `/status`. |
| macOS App Sandbox and an agent subprocess sandbox solve different problems. | Apple App Sandbox entitlement docs; Codex sandbox docs/repo. | A signed app may use App Sandbox, XPC/helper tools, and security-scoped bookmarks, but commands requested by the model still need a separate subprocess sandbox/policy. |
| Native Mac apps should pass user-selected folder access deliberately. | Apple user-selected file and security-scoped bookmark docs. | For a Claro-style app, let the user choose a workspace, store scoped access if needed, pass scoped bookmarks or data to helpers, and avoid broad Full Disk Access. |
| Context bloat is primarily tool-output bloat. | OpenAI Codex agent loop discusses prompt assembly/context; mini-SWE-agent truncates observations; OPENDEV paper focuses on compaction. | Keep full stdout/stderr in trace artifacts, but send shaped head/tail/error-focused observations to the model. Deterministic truncation before LLM summarization. |
| Aider shows that repo context should be selected, not dumped. | Aider repo map docs. | Generate lightweight repo maps/symbol maps for coding agents, but let the model request exact files lazily. Tune the token budget with evals. |
| Edit formats are model-facing APIs. | Aider edit format docs; Codex patch/application flow. | Make edits a first-class patch operation with preview, validation, rollback, and model-specific formatting support. Avoid arbitrary heredoc writes as the only edit path. |
| Interactive modes should include confirm/yolo/human or read-only/workspace-write/full-access equivalents, but unsafe modes must be visibly scoped. | mini-SWE-agent modes; Codex approval/sandbox combinations; Claude permission modes. | Let the user switch autonomy level mid-session, but display mode, cwd, writable roots, network state, and current pending action at all times. |
| Security must assume indirect prompt injection and tool misuse will happen. | Microsoft indirect prompt injection guidance; OWASP Agentic Top 10. | Isolate untrusted content, use least privilege and short-lived privileges, require human review for risky actions, log every tool decision, and test malicious repo/content scenarios. |

### Direct Facts vs. Implementation Implications

- **Direct facts from sources**: Codex App Server exposes JSON-RPC/JSONL transports and thread/turn/item primitives; Codex protocol treats UI as external to the core; Claude Code documents tiered permissions, modes, hooks, and settings precedence; mini-SWE-agent exposes `confirm`, `yolo`, and `human` modes; Aider documents repo maps and edit formats; Apple documents App Sandbox entitlements, user-selected file access, security-scoped bookmarks, child-process inheritance, and XPC as preferred privilege separation.
- **Implementation implications for a new runner**: use an event protocol, first-class patch pipeline, subprocess sandbox, deterministic policy engine, JSONL trace, and explicit workspace trust model. These are synthesized from the direct facts above; validate them with local evals and security tests before treating them as product requirements.

---

## Core Shape

A local CLI runner is not "an agent that can use a computer." It is a program that:

1. Assembles instructions, local environment context, repo docs, tool schemas, and conversation state.
2. Calls a model.
3. Receives text and/or tool calls.
4. Applies deterministic policy to each requested action.
5. Executes approved actions in a constrained local environment.
6. Feeds structured observations back to the model.
7. Persists enough trace data to resume, audit, debug, and replay.

The smallest useful runner can be a single loop plus one `bash` tool. The product-grade runner is the same loop with better boundaries around it.

```text
user prompt
  -> prompt assembler
  -> model call
  -> tool-call parser
  -> policy engine
  -> sandboxed executor
  -> observation formatter
  -> event log
  -> next model call
```

The LLM should never directly touch files, run commands, hold credentials, or decide authorization. It proposes actions; the runner executes or blocks them.

### Recommended Control Plane

Source basis: Codex App Server and protocol docs.

Build the runner around a typed bidirectional protocol, even if the first UI is only a terminal prompt.

```text
UI client (CLI/TUI/SwiftUI/IDE/test harness)
  -> initialize / configure session
  -> start turn / interrupt / approve / deny / answer question

agent core / app-server / runner
  -> turn started
  -> item started / delta / completed
  -> approval request
  -> command output chunk
  -> patch proposed / applied
  -> warning / error
  -> turn completed
```

Practical rules:

- Treat the UI as replaceable. A terminal, menu bar panel, IDE sidebar, and automated test should all drive the same core.
- Use newline-delimited JSON over stdio for the first transport. It is easy to spawn as a child process, test with fixtures, and bridge from Swift, Rust, Python, or TypeScript.
- Include client metadata and protocol version in initialization. Generate client schemas from the protocol when possible.
- Let each turn override cwd, model, sandbox policy, approval mode, and permissions profile. Store the resolved values in turn metadata.
- Support `interrupt` as a first-class operation. Local commands and model streams need cooperative cancellation and process-group termination.
- Keep a single running task per session unless the protocol explicitly represents multiple workers. For parallel work, use multiple sessions/threads so state is inspectable.

This control-plane split is the cleanest answer to "agent with UI, but local CLI-level execution": the UI renders state and gathers approvals; the runner owns model calls, command execution, patches, policy, sandboxing, and trace persistence.

---

## Minimum Viable Runner

Build this before adding subagents, GUI automation, memory systems, or custom tool marketplaces:

1. `run_one_turn(prompt, cwd)` with a fixed system prompt and one workspace.
2. One model client with streaming disabled until the loop is correct.
3. One `shell(command, cwd, timeout_ms)` tool that uses `subprocess.run` / `Command` / `Process`.
4. One edit path: either a first-class `apply_patch(patch)` tool or a model-facing patch format intercepted by the runner.
5. A simple approval gate: reads/searches allowed, writes/shell ask, destructive deny.
6. A JSONL trace with request, response, tool call, tool result, approval decision, and errors.
7. Output truncation with head/tail and byte counts.
8. A final diff summary before returning control to the user.

This MVP is enough to test whether the model can inspect a repo, edit a file, run tests, recover from errors, and stop.

---

## Prompt Assembly

Local runners win or lose on prompt assembly. Do not just send the user message.

Recommended stable prefix:

- Product/developer instructions: what the runner is, how it should communicate, how to handle risk.
- Tool definitions: stable order for cache hits and predictable model behavior.
- Sandbox/approval state: current mode, writable roots, network policy, whether escalation exists.
- Workspace instructions: `AGENTS.md`, `CLAUDE.md`, repo-specific rules, project config.
- Environment context: cwd, shell, OS, architecture, date/time, git branch/status when useful.
- User task.

Codex explicitly injects sandbox instructions, user/project instructions, and environment context before the user turn. It appends new messages when sandbox configuration or cwd changes instead of rewriting earlier prompt content, preserving prompt-cache stability.

Practical rules:

- Keep stable content before volatile conversation history.
- Load repo instructions before tool use.
- Include exact cwd and shell because generated commands depend on them.
- Include macOS-specific command differences when relevant, such as BSD `sed -i ''`.
- Prefer compressed stable docs in the prefix; use tools for dynamic files.
- Keep tool order deterministic, especially when MCP servers can change tool lists.

---

## Tool Surface

There are two viable local-runner styles.

### Shell-first

Expose one powerful `shell` tool and teach the model to use `rg`, `sed`, `awk`, `git`, package managers, test runners, and local scripts.

Strengths:
- Minimal tool schema.
- Works across languages and project types.
- Easy to implement and replay.
- Matches mini-SWE-agent and much of Codex's philosophy.

Weaknesses:
- Shell output can flood context.
- Commands need strong policy and sandboxing.
- Editing through heredocs or `sed` is less inspectable than patches.
- Untrusted text flowing into shell commands is dangerous.

Use when the model is strong, the workspace is sandboxed, and the task benefits from raw local tools.

### Primitive tools

Expose `read_file`, `list_files`, `search`, `apply_patch`, and maybe `shell`.

Strengths:
- Better output shaping and path checks.
- Easier read/write separation.
- Easier policy enforcement per operation.
- Safer for app-embedded agents.

Weaknesses:
- More active schemas compete for model attention.
- Tool design mistakes become model failure modes.
- You can accidentally build a limited fake computer worse than the real shell.

Use when you need tight control, Mac App Store-compatible UX, or non-coding automation with sensitive local data.

### Practical default

For a production local CLI, start with:

- `shell`: for repo inspection, builds, tests, package scripts, and commands that are naturally shell-native.
- `apply_patch`: first-class edit operation with diff preview and rollback.
- `read_file` / `search`: optional wrappers only if you need output limiting, gitignore-aware search, secret redaction, or better UX than shell output.

Do not add scenario-specific tools until repeated traces show the model wasting steps or causing risk that a primitive cannot handle.

---

## Shell Execution

Source basis: OpenAI local shell docs, mini-SWE-agent docs, Claude Code permissions docs, Codex approval/sandbox docs.

Prefer independent subprocess execution by default. mini-SWE-agent's docs call out why: no persistent shell prompt detection, no shell session corruption, easier sandbox substitution, and easier replay.

Recommended subprocess semantics:

```json
{
  "command": "npm test -- --runInBand",
  "cwd": "/abs/workspace",
  "timeout_ms": 120000,
  "env_policy": "sanitized",
  "sandbox": "workspace-write",
  "network": false
}
```

Implementation requirements:

- Run each command in a fresh process group/session so timeouts can kill descendants.
- Capture stdout/stderr separately, but return a merged chronological view when useful.
- Limit bytes and lines; return head/tail plus omitted byte counts.
- Strip or bound ANSI control sequences.
- Record exit code, signal, duration, cwd, sandbox mode, approval mode, and timeout flag.
- Set deterministic env defaults: `PAGER=cat`, `MANPAGER=cat`, `CI=1` when appropriate, disabled progress bars where possible.
- Never inherit all user env vars blindly into agent-visible output.
- Redact secrets from command output before returning it to the model.
- Deny or ask before running commands that use network, mutate outside workspace, install hooks, modify login shell files, alter credentials, or run privileged commands.

Fresh-process execution does not mean the agent cannot use state. It means the state must be explicit: prefix commands with `cd`, use files, use venv paths, or use named background sessions.

Command classification guidance from Claude Code permissions:

- Built-in read-only families (`ls`, `cat`, `head`, `tail`, `grep`, `find`, `wc`, `diff`, `stat`, `du`, `cd`, read-only `git`) are candidates for auto-approval, but only after argument parsing and cwd/path checks.
- Treat wrappers (`npx`, `devbox run`, `mise exec`, `docker exec`, `watch`, `setsid`, `flock`, `find -exec`) as executing another command. Exact allow rules are safer than prefix allow rules.
- URL/domain restrictions inside arbitrary Bash are weak because options, redirects, variables, and spacing can evade naive pattern rules. Prefer a controlled fetch tool or proxy when network access is required.
- File-tool deny rules do not automatically protect against `cat .env` through Bash. Enforce sensitive path denial in the subprocess sandbox and command policy too.

---

## PTY and Long-Running Processes

Source basis: Codex TUI source dependencies, Ratatui/Bubble Tea terminal lifecycle docs, general terminal process behavior.

Avoid a persistent interactive shell as the default. Add PTY support only for commands that really need it:

- dev servers
- REPLs
- interactive installers
- test watch modes
- terminal UIs
- programs whose behavior changes without a TTY

PTY/session tool design:

```json
{
  "session_id": "devserver-1",
  "command": "npm run dev",
  "cwd": "/abs/workspace",
  "timeout_ms": 1000,
  "background": true
}
```

Required operations:

- `start_session`
- `write_stdin`
- `read_output`
- `send_signal`
- `close_session`

Rules:

- Keep a bounded ring buffer per session.
- Surface whether the process is still running.
- Support idle timeout and max lifetime.
- Do not let the model rely on hidden cwd/env changes in a session unless each session state is visible in observations.
- Kill process groups, not just parent PIDs.
- On macOS, account for GUI-launched apps having different PATH/env than login shells.

The model should see session state as data, not infer it from terminal behavior.

---

## Patch and Edit Pipeline

Editing is where local runners cause the most real damage. Prefer a patch tool over arbitrary file writes.

Patch tool requirements:

- Parse a constrained patch grammar.
- Reject absolute paths unless your policy explicitly allows them.
- Normalize paths and prevent `..` escape.
- Block writes to `.git`, `.env*`, credentials, keychains, shell startup files, and runner config by default.
- Apply against the current file contents, not stale model assumptions.
- Fail with actionable errors: file missing, context not found, multiple matches, binary file, permission denied.
- Show a diff preview before approval in interactive modes.
- Record inverse patch or snapshot for rollback.
- Run formatters/tests after apply only through normal policy.

Codex uses a model-facing patch format and instructs the model to use `apply_patch`; if you control the runner API, make patch application a first-class tool rather than a shell command. If you keep it as a shell-like command for model compatibility, intercept it before it reaches the real shell.

For app-embedded agents, consider a stricter edit path:

- `propose_patch` generates a patch artifact.
- user or policy approves it.
- `apply_patch` writes it.
- `verify_patch` computes diff and optional test result.

---

## macOS Sandboxing

Source basis: Apple App Sandbox entitlement docs, Apple helper/XPC guidance, Codex sandbox/approval docs, Codex repository, OpenAI GPT-5.2-Codex system card addendum for macOS Seatbelt mention.

For a local macOS CLI, the practical sandbox boundary is process-level sandboxing around spawned commands, plus filesystem policy in the runner.

Codex docs define `read-only`, `workspace-write`, and `danger-full-access` sandbox modes, with network controlled separately in workspace-write. OpenAI's Codex security materials and repository/issues identify macOS enforcement as Apple's Seatbelt / `/usr/bin/sandbox-exec`; treat that implementation detail as version-sensitive and verify in the current Codex source before copying it.

Useful modes:

- `read-only`: commands can inspect but not write; good for planning, review, and untrusted repos.
- `workspace-write`: commands can write within the approved workspace; network off by default.
- `danger-full-access`: no sandbox; use only inside an external sandbox such as a disposable VM/devcontainer.

macOS-specific implementation notes:

- `/usr/bin/sandbox-exec` is the common wrapper for Seatbelt profiles, but it is not a friendly high-level API. Treat profile generation as security-sensitive code with tests.
- Use OS sandboxing for spawned commands, but still enforce runner policy before process launch. The sandbox is the last line of defense, not the permission system.
- Deny network by default. If network is enabled, prefer domain allowlists or a controlled proxy.
- Mount/allow only the workspace and required temp dirs for writes.
- Hide or deny reads to secrets: `.env*`, `.ssh`, `.aws`, `.gnupg`, keychain material, browser profiles, private app data, and runner credentials.
- Keep `.git` at least write-protected unless the user explicitly asks for git mutations.
- Detect sandbox failure as a hard error. Do not silently fall back to unsandboxed execution.

If this runner is embedded in a signed macOS app, distinguish two sandboxes:

- App Sandbox entitlements constrain the app itself.
- A subprocess sandbox constrains model-generated commands.

Do not assume the App Sandbox alone gives a good agent sandbox. A coding agent often needs controlled access to user-selected project folders, subprocess execution, and local tools. Keep those capabilities behind explicit user selection, scoped bookmarks/permissions where applicable, and a separate tool policy.

### App-Embedded Mac Runner Patterns

Source basis: Apple App Sandbox entitlement reference.

Use one of these topologies:

1. **Standalone CLI**: simplest for developer tools. It runs outside App Sandbox unless manually sandboxed per command. Good for prototypes and internal power-user workflows.
2. **Sandboxed app + embedded helper tool**: the GUI owns user consent and launches a bundled command-line helper. Apple notes this is sometimes useful, but XPC is often a better choice for separate-process code.
3. **Sandboxed app + XPC service/helper**: preferred when you need privilege separation, a stable IPC contract, or clearer process boundaries inside a Mac app.
4. **Sandboxed app + external user-installed CLI**: GUI discovers and drives a separately installed binary. Good for fast iteration, weaker for App Store-style packaging.

Mac-specific constraints:

- App Sandbox removes most system interaction by default and restores capabilities through entitlements. Do not ask for broad entitlements when user-selected file access or bookmarks are enough.
- `com.apple.security.files.user-selected.read-write` grants access to files/folders explicitly selected through standard dialogs. Persistent access should use security-scoped bookmarks.
- Child-process sandbox inheritance (`com.apple.security.inherit`) only inherits static app entitlements; Apple recommends XPC for privilege separation and notes post-launch PowerBox access is not inherited automatically.
- Network, microphone, Accessibility, Apple Events, pasteboard, and file access are separate user trust domains. Do not collapse them into a single "agent mode" permission.
- For a Claro-style product, dictation/paste should remain separate from project automation. Agent execution should be an explicit, scoped action with a visible workspace and rollback story.

---

## Workspace Trust and Config

Source basis: Codex config docs, Claude Code settings docs, OWASP Agentic Skills guidance.

Local agents read instructions from the same repo they may act on. That is useful and dangerous.

Recommended loading order:

1. User/global config.
2. System/admin requirements.
3. Trust decision for the workspace.
4. Project-local config, hooks, rules, skills, and MCP servers only after trust.
5. Repo instruction files scoped by directory.

Rules:

- Never let untrusted project config grant itself permissions.
- Treat repo docs, issues, READMEs, and source comments as untrusted task context unless they are explicit user/developer instructions.
- Keep user/global deny rules above project allow rules.
- Make config precedence inspectable via a `/status` command.
- Persist workspace trust decisions separately from repo files.
- Include active sandbox, approval mode, writable roots, network state, and loaded config sources in every trace.

Codex docs explicitly load project config only when a project is trusted. Claude Code docs support deny rules for sensitive files. Use those as baseline patterns.

Treat repo-local instructions as a supply-chain surface:

- Project config may affect permissions, hooks, skills, MCP servers, memory files, and command behavior.
- Project-local hooks/scripts run with user credentials once enabled.
- A malicious repo can try to smuggle instructions through README, tests, tool output, generated docs, or config.
- Therefore, trust is not just "can read this folder"; it is "can this folder influence execution policy?"

Recommended trust levels:

| Level | Project instructions | Project config/hooks/MCP | Writes | Network |
|-------|----------------------|--------------------------|--------|---------|
| untrusted | read as untrusted context only | disabled | no | no |
| trusted-readonly | load instructions | disabled by default | no | no |
| trusted-workspace | load instructions | ask before enabling | workspace only | ask/deny default |
| trusted-automation | load instructions | allowed by policy | workspace + explicit roots | explicit allowlist |

---

## Permission Policy

Source basis: Codex approvals/security docs, Claude permissions docs, Microsoft indirect prompt injection guidance, OWASP Agentic Top 10.

Approvals are UX; policy is security. Build both.

Policy inputs:

- tool name
- command/patch arguments
- cwd and normalized paths
- sandbox mode
- network state
- workspace trust
- current task contract
- untrusted content influence
- configured allow/deny rules
- user approval mode

Decision shape:

```json
{
  "decision": "allow | ask | deny",
  "reason": "network access requested in workspace-write mode",
  "preview": "curl https://example.com",
  "risk": "network"
}
```

Suggested defaults:

- Allow: read-only commands inside workspace, `rg`, `ls`, `git status`, `git diff`, `xcodebuild -showBuildSettings` when output is bounded.
- Ask: file writes, package installs, test commands that may run project scripts, git mutations, process launches, network.
- Deny: secrets access, credential stores, destructive filesystem commands outside workspace, privilege escalation, persistence changes, remote exfiltration, system settings changes.

Approval fatigue is real. Reduce prompts by tightening the sandbox and allowlisting safe command families, not by bypassing policy.

Use deterministic precedence:

1. Hard denies: secrets, protected paths, privilege escalation, persistence changes, destructive outside workspace.
2. Workspace trust: untrusted repos cannot load or grant execution policy.
3. Sandbox capability: if the OS/process sandbox cannot enforce the requested mode, fail closed.
4. Tool/command classifier: read-only vs write/network/exec/destructive.
5. User/admin allow rules.
6. Runtime approval.
7. Optional reviewer/critic agent for already-required approvals, never as the only gate.

The model may explain a request, but it should not be the final authority for whether it can run.

---

## Hooks

Source basis: Claude Code hooks docs and settings docs.

Hooks are runner extension points. Claude Code supports lifecycle/tool hooks that can allow, deny, add context, run formatters, collect logs, or spawn verification agents.

Useful hook events:

- `SessionStart`: inspect repo, set env, add local context.
- `PreToolUse`: block risky commands, enforce custom policy, redact args.
- `PostToolUse`: run formatter, collect diff summary, index changed files.
- `FileChanged`: trigger lightweight diagnostics.
- `CwdChanged`: update environment context.
- `SessionEnd`: write summary, cleanup processes, persist metrics.

Rules:

- Hooks execute with user credentials. They are not a trust boundary.
- Run hooks with timeouts and bounded output.
- Log hook decisions and failures.
- Do not load project hooks before workspace trust.
- Keep hook output explicit: `{ decision, reason, additionalContext }`.

Hooks are best for local customization and team policy, not for core safety. Core allow/deny checks should live in the runner.

---

## Event Log and Replay

Use append-only JSONL as the source of truth. Do not rely only on the model-visible conversation.

Minimum event types:

- `session_started`
- `user_message`
- `prompt_assembled`
- `model_request_started`
- `model_response_completed`
- `tool_call_requested`
- `policy_decision`
- `approval_requested`
- `approval_decided`
- `command_started`
- `command_output_chunk`
- `command_exited`
- `patch_proposed`
- `patch_applied`
- `diff_snapshot`
- `context_compacted`
- `error`
- `session_ended`

Each event should include:

- session id
- monotonic sequence number
- timestamp
- agent version/git SHA
- model id/provider
- prompt/tool schema version
- cwd
- sandbox/approval mode
- redaction status

Replay modes:

- `replay-transcript`: reconstruct model-visible messages.
- `replay-tools`: re-run tool calls in a disposable workspace.
- `inspect`: browse actions, diffs, outputs, and approvals.

SWE-agent and mini-SWE-agent emphasize trajectories for debugging. Codex and Claude Code-style systems benefit from the same event-sourcing pattern because long local sessions otherwise become impossible to explain.

---

## Context Handling for CLI Agents

Source basis: Aider repo maps, SWE-agent ACI notes, mini-SWE-agent observation templates, Codex agent loop, OPENDEV preprint.

Shell output is the main source of context rot. A local runner must shape observations aggressively.

Observation formatter rules:

- Return exit code, duration, cwd, and command.
- Show short output inline.
- For long output, show head and tail with omitted byte/line count.
- Preserve error lines, stack traces, failing tests, and file paths.
- Suggest next targeted commands when truncation hides important detail.
- Store full output in the event log, but do not always send it to the model.
- Redact secrets before both model context and logs unless secure full-trace retention is explicitly configured.

Do not summarize command output with an LLM as the first response. Try deterministic truncation, masks, and slices first.

Context layers for local coding agents:

1. **Stable prefix**: product/system/developer instructions, tool contracts, policy state, environment summary.
2. **Workspace instructions**: trusted `AGENTS.md`/`CLAUDE.md`-style files, scoped by directory and source.
3. **Repo map**: symbols/files/dependencies selected to a token budget, similar to Aider's repo map pattern.
4. **Active files**: exact file excerpts loaded by tools, not by eager dumping.
5. **Recent observations**: latest command results and patch outcomes.
6. **Trace references**: full logs stored outside context and addressable by artifact id/path.

Useful observation format:

```json
{
  "command": "xcodebuild test -scheme Claro",
  "cwd": "/abs/workspace",
  "exit_code": 65,
  "duration_ms": 42133,
  "output_head": "...",
  "output_tail": "...",
  "omitted_bytes": 183422,
  "detected_failures": [
    {"file": "ClaroTests/AgentPolicyTests.swift", "line": 42, "message": "expected deny"}
  ],
  "artifact": "traces/session-123/commands/0007.log"
}
```

---

## Terminal UX

Source basis: Codex TUI/App Server docs and source comments, mini-SWE-agent interactive modes, `Terminal Is All You Need`, Ratatui/Bubble Tea docs.

A useful local agent is a process supervisor and a collaboration UI.

CLI/TUI should show:

- current mode: planning, read-only, workspace-write, full access
- cwd and git branch
- active model
- current tool call
- command stdout/stderr tail
- pending approval with exact command/patch/diff preview
- token/cost/time budget
- background process list
- final changed files and verification status

Required controls:

- interrupt current command
- deny with reason
- approve once
- approve matching rule for session
- switch read-only/workspace-write/full-access
- show `/status`
- show `/diff`
- rollback last patch when possible
- save/exit and resume

Avoid hiding tool details. Trust comes from showing what happened and what will happen next.

TUI/application architecture:

- Keep a single state model fed by runner events.
- Render committed transcript items separately from in-flight streaming/tool state.
- Make approvals modal but inspectable: exact command, cwd, sandbox, env delta, paths, network target, and diff preview.
- Preserve terminal health: raw mode and alternate screen must be restored on crash/exit; test with panic/error paths.
- Add `/status`, `/diff`, `/permissions`, `/trace`, `/resume`, and `/rollback` early. They are debugging tools as much as UX.
- If a GUI wraps the runner, show the same primitives in native controls rather than inventing a hidden magical mode.

---

## macOS Integration Boundaries

If the runner controls more than files and shell, separate capability domains:

- Files and shell: local project automation.
- Accessibility: reading/clicking/typing in apps through AX APIs.
- Apple Events: app scripting.
- Pasteboard: moving generated text into the active app.
- Network: external APIs and web access.
- Credentials: keychain/API token access.

Each domain needs a separate permission story. Do not treat Accessibility permission as a generic "agent can use my Mac" switch. For user trust, expose concrete action previews: target app, text to paste, command to run, file to edit, URL to fetch.

For products like Claro, dictation should remain the default low-friction path. Agentic local automation should be explicit, reversible, and visibly scoped.

---

## Implementation Stack Choices

### Python

Best for prototypes and research:

- `subprocess.run`
- `pty` / `asyncio.subprocess` for long-running commands
- `rich` / `textual` for TUI
- `pydantic` for schemas
- `jsonlines` traces

Risk: packaging and native macOS distribution require extra work.

### TypeScript/Node

Best for SDK-rich CLI tools:

- `child_process.spawn`
- `node-pty` when needed
- Zod/JSON Schema for tools
- Ink/Blessed for TUI
- strong ecosystem for MCP and AI SDKs

Risk: subprocess and signal behavior needs careful testing across shells.

### Rust

Best for a durable cross-platform CLI:

- fast startup
- single binary
- strong process control
- good fit for sandbox wrappers and patch parsers

Risk: slower iteration and more code for provider SDKs/TUI.

### Swift/macOS app plus helper

Best when the agent is embedded in a native Mac product:

- SwiftUI/AppKit for UI
- XPC/helper process for execution boundary
- Security-scoped bookmarks for user-selected folders
- Keychain for credentials
- native permission UX

Risk: subprocess sandboxing, distribution, and TCC permissions are more complex than a standalone CLI.

---

## File Layout

For a small but durable runner:

```text
runner/
  bin/agent
  core/
    loop.*
    prompt_assembler.*
    model_client.*
    events.*
    context_window.*
  tools/
    shell.*
    apply_patch.*
    read_file.*
    search.*
  policy/
    permissions.*
    path_policy.*
    command_classifier.*
    redaction.*
  sandbox/
    macos_seatbelt.*
    linux_bwrap.*
    no_sandbox.*
  ui/
    tui.*
    approvals.*
    diff_view.*
  tests/
    fixtures/
    policy_tests.*
    patch_tests.*
    replay_tests.*
```

Keep the shell executor, patch applier, and policy engine testable without the model.

---

## Test Harness

Source basis: SWE-agent trajectories/evals, mini-SWE-agent trajectory output, OWASP/Microsoft security guidance, Codex approval/sandbox modes.

Do not evaluate a local runner only by watching demos.

Unit tests:

- path normalization and symlink escape
- secret path deny rules
- command classifier
- sandbox profile generation
- patch parser/apply/rollback
- output truncation/redaction
- approval precedence

Integration tests:

- inspect repo, edit file, run test, report diff
- denied secret read
- denied network command
- workspace-write cannot write outside workspace
- background process starts and stops
- interrupted command leaves runner healthy
- resume from JSONL trace

Security tests:

- repo README instructs agent to read `.env`
- test output contains fake secret
- malicious package script tries `curl`
- patch attempts `../outside`
- symlink points outside workspace
- project config attempts to loosen permissions before trust
- project hook attempts to execute before trust
- "safe" grep command uses an unsafe flag or command wrapper
- network command hides target in an environment variable
- model asks for Accessibility/pasteboard/network while task only requires file edits
- long command output buries the actual failure outside the first/last chunk

Eval tasks:

- small bug fix
- refactor with tests
- docs-only edit
- build failure diagnosis
- command failure recovery
- user rejection handling

Grade outcomes, policy decisions, and trace quality.

Minimum release gates:

- Every command/patch in a trace has a policy decision and source.
- A denied operation returns an actionable observation to the model without leaking secrets.
- Sandbox failure fails closed and is visible in UI/logs.
- Approval prompts identify whether the requested permission is one-time, session-scoped, or persistent.
- Resume/replay can reconstruct what the model saw and what the runner actually executed.
- macOS app builds never require broad TCC permissions unless the feature truly uses that domain.

---

## Design Heuristics

- Start shell-first, then add primitives only where traces prove they help.
- Use subprocesses for routine commands; reserve PTYs for true interactive sessions.
- Make patches first-class because edits deserve preview, rollback, and policy.
- Put sandbox state in both policy and prompt; enforce it only in code.
- Keep full traces outside context; send shaped observations to the model.
- Treat every repo file, web page, and command output as untrusted content.
- Ask less by sandboxing better, not by trusting the model more.
- Prefer explicit cwd/env/session state over hidden mutable terminal state.
- Build replay before tuning prompts; otherwise failures are anecdotes.
- On macOS, separate "can run commands" from Accessibility, pasteboard, Apple Events, network, and credentials.

---

## Red Flags

- The runner silently falls back to unsandboxed execution.
- Project-local config loads before workspace trust.
- The model can write `.git`, `.env`, shell startup files, or runner config by default.
- Tool results return unbounded command output.
- Permission decisions are made only by the model.
- A persistent shell session is the only execution path.
- Hooks can override deny rules.
- Network is enabled by default.
- The runner cannot replay a failed session.
- The final answer says files changed but there is no diff snapshot.

---
name: amplifier-skill
description: Delegate complex work to Microsoft Amplifier from AI agents (Manus, Claude Code, Cursor, Copilot). Use when users explicitly ask to use Amplifier, delegate to Amplifier, inspect Amplifier history/context, or discover Amplifier agents and bundles. Prefer conservative delegation and use CLI-first discovery with filesystem fallbacks in restricted environments.
---

# Amplifier Skill

Delegate only on explicit Amplifier intent and keep simple local work local.

## Platform Context
- **Manus:** Autonomous sandbox agent. Run Amplifier non-interactively (`amplifier run`). Use `export PATH="$HOME/.local/bin:$PATH"` before commands.
- **Claude Code / Cursor / Copilot:** Interactive or semi-interactive. Use standard commands.
- **Memory / Context:** If asked to maintain long-term state, use the `memory` bundle (`amplifier run --bundle memory`) to enable MemPalace and `project-context` integrations.

## Workflow

### 1. Run preflight checks

Verify Amplifier is available before delegating (ensure `uv` path is set for sandboxes):

```bash
export PATH="$HOME/.local/bin:$PATH"
command -v amplifier
amplifier provider current
```

If provider state is missing or invalid, instruct:
```bash
amplifier init
```
(Or configure `~/.amplifier/settings.yaml` manually if in a non-interactive sandbox like Manus).

### 2. Decide whether to delegate

Delegate when the user is explicit about Amplifier, for example:
- "use Amplifier"
- "delegate to Amplifier"
- "show Amplifier context/history"
- "what agents does Amplifier have?"

Keep local when the task is a quick edit, simple shell command, obvious fix, or routine repo search.

When intent is ambiguous, ask one short clarification instead of auto-delegating.

### 3. Delegate tasks to Amplifier

Use single-shot delegation (Required for autonomous agents like Manus):

```bash
amplifier run "<task description>"
```

Use multi-turn delegation (For interactive terminals like Claude Code):

```bash
amplifier
amplifier continue
amplifier session resume <session-id>
```

Use optional targeting only when the user asks:

```bash
amplifier run --bundle <bundle-name> "<task description>"
amplifier run --provider <provider-name> "<task description>"
```

### 4. Query session context safely

Use the helper script first:

```bash
./scripts/session_context.sh --project "$PWD" --limit 10
./scripts/session_context.sh --all-projects --limit 10
```

The script uses `amplifier session list` first, then falls back to
`~/.amplifier/projects/**/metadata.json` parsing when runtime access is blocked.

### 5. Discover agents without hardcoding catalogs

Use the helper script first:

```bash
./scripts/list_agents.sh
./scripts/list_agents.sh --bundle foundation
```

The script tries runtime commands first, then parses cached manifests under
`~/.amplifier/cache/*/bundle.md`.

For bundle-level inspection:

```bash
amplifier bundle list
amplifier bundle current
amplifier bundle show <bundle-name>
```

### 6. Handle failure modes

If `amplifier` is missing, instruct installation:

```bash
uv tool install git+https://github.com/microsoft/amplifier
```

If the task requires TDD, design spec validation, or plan generation, use the `superpowers` bundle (`amplifier run --bundle superpowers`).

If runtime commands fail in sandboxed/restricted sessions, keep moving with the helper script fallbacks and summarize what data was available.

If no sessions or agents are found, return a clear empty-state response and next command to run.

## Guardrails

- Delegate conservatively: explicit intent only.
- Avoid stale hardcoded model, bundle, or agent lists.
- Prefer dynamic discovery over static documentation snapshots.
- Summarize Amplifier output and next steps after delegation completes.

# 07 - Configuring OpenCode for Real Work<no value>

In this post, we will move beyond OpenCode's default settings and configure it for actual development work using local
models.

## Why

Local models require different handling than cloud providers because context length directly impacts response speed on
your hardware. If we don't configure OpenCode correctly, response times will balloon as the codebase grows, making the
tool unusable. We need a setup that balances project awareness with interactive speed.

---

## Context Length: The Critical Difference

With cloud providers like Claude or ChatGPT, context length is nearly invisible—you send 200K tokens and get fast
responses. We need to understand that **local setups behave differently**, and managing this is essential.

### Token Generation (TG) Decreases with Context Fill

As the context window fills up, your generation speed drops. On my Minisforum MS-S1 Max (Strix Halo, 128GB), the
benchmark data shows this clearly:

| Model                 | TPS @ 5% Fill | TPS @ 75% Fill | Retention |
| --------------------- | ------------- | -------------- | --------- |
| qwen3-coder-next Q4   | ~43           | ~30            | 70%       |
| qwen3.5-35b-a3b (MoE) | ~48           | ~34            | 71%       |
| qwen3.5-27b (Dense)   | ~12           | ~9             | 74%       |

I prefer Mixture-of-Experts (MoE) models like **qwen3.5-35b-a3b** because they maintain similar speed retention to dense
models while being significantly faster. In my tests, the 35B-A3B generates 4x more tokens per second than the 27B dense
model.

### Prompt Processing (PP) Increases with Context Fill

This is the real performance killer. Your Time to First Token (TTFT) scales sharply as you fill the context window:

| Model         | TTFT @ 5% (6k tokens) | TTFT @ 75% (98k tokens) |
| ------------- | --------------------- | ----------------------- |
| 35B-A3B Q4    | ~8s                   | ~86s                    |
| Coder-Next Q4 | ~10s                  | ~101s                   |

### Strategic Context Management

Don't assume more context is always better. Based on my benchmarks, we should follow these practical limits for a
responsive workflow:

- **Comfortable**: TTFT < 30s
- **Usable**: TTFT < 120s (2 minutes)
- **Marginal**: TTFT < 5 minutes

**Recommended context lengths for 35B-A3B Q4:**

| Performance Tier | Context Limit | Resulting TTFT |
| ---------------- | ------------- | -------------- |
| Comfortable      | ~30K tokens   | ~30s           |
| Usable           | ~65K tokens   | ~66s           |
| Marginal         | ~131K tokens  | ~85s\*         |

_\*Note: TTFT can fluctuate at extreme context fills due to KV cache offloading, but expect ~90s for full windows._

---

## How to Configure the Environment

### 1. Optimize Inference in LM Studio

OpenCode does not support inference settings in its own config schema, so we must set these directly in LM Studio. Use
these values to balance creativity and logic:

- **Temperature**: 0.6
- **Top_P**: 0.95
- **Top_K**: 40
- **Presence/Frequency Penalty**: 0.0

### 2. Define the Workspace

OpenCode reads `AGENTS.md` to understand your project. Without it, the model has no context regarding your structure or
tech stack. Create this file in your root directory:

```markdown
# Project Context

## Structure

- /src - Source code
- /tests - Test files

## Tech Stack

- Flutter
- BloC

## Commands

- fvm flutter pub get
- fvm flutter build
```

### 3. Use LSP Servers

LSP (Language Server Protocol) servers are essential. They provide semantic understanding to the model without consuming
your precious context budget. Ensure your local environment has the relevant LSPs installed for your language (e.g.,
`dart-sdk` for Flutter).

### 4. Define Agents

Agents are specialized AI assistants configured for specific tasks. OpenCode ships with two built-in primary agents:

- **Build** — Default agent with all tools enabled for full development work
- **Plan** — Restricted agent for analysis and planning; defaults to `ask` for file edits and bash commands

You can create custom agents in `opencode.json` or as markdown files in `.opencode/agents/`. Each agent can have its own
model, temperature, tools, and permissions.

```json
{
  "agent": {
    "review": {
      "description": "Reviews code for best practices",
      "mode": "subagent",
      "tools": {
        "write": false,
        "edit": false
      }
    }
  }
}
```

### 5. Create Skills

Skills let OpenCode discover reusable instructions on-demand. They live in `SKILL.md` files within skill folders:

- Project: `.opencode/skills/<name>/SKILL.md`
- Global: `~/.config/opencode/skills/<name>/SKILL.md`

Each skill requires frontmatter with `name` and `description`:

```markdown
---
name: flutter-bloc
description: Guidelines for Flutter BloC architecture
---

## When to use

Use this when working with state management in Flutter apps.

## Guidelines

- Separate events, states, and blocs into different files
- Use Equatable for state comparison
- Emit new states instead of mutating existing ones
```

The agent loads skills via the native `skill` tool when needed. This keeps your context clean—the skill content only gets
added when explicitly invoked.

### 6. Define Custom Commands

Commands let you create reusable prompts accessible via `/command-name`. Define them in `.opencode/commands/` or via
`opencode.json`:

```markdown
---
description: Run tests with coverage
agent: build
---

Run the full test suite with coverage report. Focus on failing tests and suggest fixes.
```

Commands support arguments (`$ARGUMENTS`, `$1`, `$2`), shell output injection (`` !`command` ``), and file references
(`@filename`):

```markdown
---
description: Review component
---

Review the component in @src/components/Button.tsx and suggest improvements.
```

### 7. Configure Rules (AGENTS.md)

OpenCode reads `AGENTS.md` for project-specific instructions. Run `/init` to auto-generate one, or create it manually:

```markdown
# Project Context

## Structure

- /src - Source code
- /tests - Test files

## Tech Stack

- Flutter
- BloC

## Guidelines

- Use conventional commits
- Always write tests for new features
```

Global rules live in `~/.config/opencode/AGENTS.md`. OpenCode also supports Claude Code compatibility—`CLAUDE.md` works as
a fallback if `AGENTS.md` doesn't exist.

---

> The examples above are simplified. We'll cover real-world configurations for Flutter development later in this guide.

## Key Points Recap

1. **Context length kills local speed** — Expect TPS to drop and TTFT to rise as you add more code to the session.
2. **MoE models are the speed kings** — qwen3.5-35b-a3b is 4x faster than its dense 27B counterpart while maintaining ~
   71%
   speed retention.
3. **TTFT scales sharply** — Moving from 5% to 75% context fill can increase your wait time from 8s to over 80s.
4. **Target 30K for speed, 65K for depth** — Keep context at 30K for quick iterations and only push to 65K when the model
   needs to "see" a large portion of the codebase.
5. **Set inference in LM Studio** — Use Temperature 0.6 and Top_P 0.95, as OpenCode won't override these.
6. **Always include AGENTS.md** — This is the model's map; without it, the LLM is flying blind in your directory
   structure.
7. **LSPs are "free" context** — Use them to provide semantic data without eating into your token limit.
8. **Instructions are commands** — When prompting OpenCode, use direct imperative verbs to get better results.
9. **Agents handle different workflows** — Use Build for full development, Plan for read-only analysis.
10. **Skills load on-demand** — Keep reusable instructions in `.opencode/skills/` and load them only when needed.
11. **Commands automate repetitive prompts** — Create `/test` or `/review` commands in `.opencode/commands/` for common tasks.
12. **Rules cascade** — Project `AGENTS.md` overrides global rules, which override Claude Code compatibility.

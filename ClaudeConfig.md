# Claude Code Full Setup Configuration

> Comprehensive setup document for replicating this Claude Code environment on a fresh machine.
> Generated 2026-02-14. No secrets, keys, or PII included — placeholders marked with `<REDACTED>`.

---

## Table of Contents

1. [System Prerequisites](#1-system-prerequisites)
2. [Claude Code Installation](#2-claude-code-installation)
3. [Directory Structure](#3-directory-structure)
4. [Base Settings — settings.json](#4-base-settings--settingsjson)
5. [Session Settings — settings.local.json](#5-session-settings--settingslocaljson)
6. [Global Instructions — CLAUDE.md](#6-global-instructions--claudemd)
7. [Project-Level Instructions — ~/CLAUDE.md](#7-project-level-instructions--claudemd)
8. [Hook Scripts](#8-hook-scripts)
9. [Custom Agents](#9-custom-agents)
10. [Plugins](#10-plugins)
11. [Memory System](#11-memory-system)
12. [Gate 3 Templates](#12-gate-3-templates)
13. [Custom Tools — ~/claude-tools/](#13-custom-tools--claude-tools)
14. [QMD Local Search Engine](#14-qmd-local-search-engine)
15. [Auto Memory — Project Memory](#15-auto-memory--project-memory)
16. [Verification Checklist](#16-verification-checklist)

---

## 1. System Prerequisites

Install these before configuring Claude Code:

```bash
# Homebrew (macOS)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Node.js 20+
brew install node@20

# pnpm
npm install -g pnpm

# jq (required by hook scripts)
brew install jq

# terminal-notifier (macOS desktop notifications)
brew install terminal-notifier

# Bun (for qmd)
curl -fsSL https://bun.sh/install | bash

# AWS CLI (if deploying to AWS)
brew install awscli

# Docker (if building containers)
brew install --cask docker

# GitHub CLI
brew install gh
gh auth login

# Codex CLI (OpenAI — Gate 3 cross-model review)
npm install -g @openai/codex

# Gemini CLI (Google — Gate 3 cross-model review)
npm install -g @anthropic-ai/gemini  # or install per Google's docs
# Note: Version should be 0.28.0+

# QMD (local markdown search engine via MCP)
bun install -g qmd

# Happy Coder (optional — commit co-authorship)
npm install -g happy-coder
```

### PATH Configuration

Add to `~/.zshrc` or `~/.zprofile`:

```bash
export PATH="$HOME/.npm-global/bin:/opt/homebrew/opt/node@20/bin:$HOME/.bun/bin:$HOME/bin:$PATH"
```

### npm Global Directory

```bash
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
```

---

## 2. Claude Code Installation

```bash
# Install Claude Code CLI
npm install -g @anthropic-ai/claude-code

# Verify
claude --version  # Should be v2.1.42+
```

---

## 3. Directory Structure

Create all required directories:

```bash
# Claude Code config
mkdir -p ~/.claude/hooks
mkdir -p ~/.claude/agents
mkdir -p ~/.claude/projects/-Users-$(whoami)/memory

# Memory system
mkdir -p ~/claude-memory/global
mkdir -p ~/claude-memory/engine
mkdir -p ~/claude-memory/projects

# Custom tools
mkdir -p ~/claude-tools
```

---

## 4. Base Settings — settings.json

File: `~/.claude/settings.json`

```json
{
  "env": {
    "CLAUDE_CODE_EFFORT_LEVEL": "medium"
  },
  "permissions": {
    "allow": [
      "Bash(pnpm typecheck)",
      "Bash(pnpm lint)",
      "Bash(pnpm lint:fix)",
      "Bash(pnpm build)",
      "Bash(pnpm db:generate)",
      "Bash(git status*)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(git branch*)",
      "Bash(git add *)",
      "Bash(node ~/claude-memory/*)",
      "Bash(ls *)",
      "Bash(wc *)"
    ]
  },
  "model": "opus",
  "enabledPlugins": {
    "frontend-design@claude-plugins-official": true,
    "context7@claude-plugins-official": true,
    "ralph-loop@claude-plugins-official": true,
    "code-review@claude-plugins-official": true,
    "playwright@claude-plugins-official": true,
    "security-guidance@claude-plugins-official": true,
    "superpowers@claude-plugins-official": true,
    "everything-claude-code@everything-claude-code": true,
    "code-simplifier@claude-plugins-official": true
  },
  "outputStyle": "clean-code",
  "spinnerVerbs": {
    "mode": "replace",
    "verbs": [""]
  },
  "skipDangerousModePermissionPrompt": true
}
```

---

## 5. Session Settings — settings.local.json

File: `~/.claude/settings.local.json`

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "70",
    "CLAUDE_CODE_EFFORT_LEVEL": "high"
  },
  "permissions": {
    "allow": [
      "Bash(find:*)",
      "Bash(grep:*)",
      "Bash(curl:*)",
      "Bash(cd:*)",
      "Bash(lsof:*)",
      "Bash(git checkout:*)",
      "Bash(git pull:*)",
      "Bash(git stash:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git reset:*)",
      "Bash(git push:*)",
      "Bash(gh pr merge:*)",
      "Bash(git status:*)",
      "Bash(git clone:*)",
      "Bash(gh pr view:*)",
      "Bash(gh pr checks:*)",
      "Bash(gh pr create*)",
      "Bash(gh run view:*)",
      "Bash(du:*)",
      "Bash(python3:*)",
      "Bash(node:*)",
      "Bash(npx tsc:*)",
      "Bash(npx prisma db push)",
      "Bash(npx prisma validate:*)",
      "Bash(npx dotenv:*)",
      "Bash(npx eslint:*)",
      "Bash(npm install:*)",
      "Bash(pnpm add:*)",
      "Bash(pnpm install:*)",
      "Bash(pnpm --filter *)",
      "Bash(chmod:*)",
      "Bash(brew list:*)",
      "Bash(brew install:*)",
      "Bash(brew search:*)",
      "Bash(jq:*)",
      "Bash(ls:*)",
      "Bash(wc:*)",
      "Bash(crontab:*)",
      "Bash(test:*)",
      "WebSearch",
      "mcp__plugin_context7_context7__resolve-library-id",
      "mcp__plugin_context7_context7__query-docs",
      "mcp__plugin_playwright_playwright__browser_click"
    ],
    "defaultMode": "acceptEdits"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-destructive.sh",
            "timeout": 5,
            "statusMessage": "Safety check..."
          }
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-patterns.sh",
            "timeout": 5,
            "statusMessage": "Checking patterns..."
          }
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "git rev-parse --is-inside-work-tree &>/dev/null && but claude pre-tool || true",
            "timeout": 5
          }
        ]
      },
      {
        "matcher": "Write|Edit|Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/observe-learning.sh pre",
            "timeout": 5,
            "statusMessage": "Observing..."
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/pre-commit-gate.sh",
            "timeout": 5,
            "statusMessage": "Gate 2 pre-commit check..."
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "git rev-parse --is-inside-work-tree &>/dev/null && npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null || true",
            "timeout": 15,
            "statusMessage": "Formatting..."
          }
        ]
      },
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "git rev-parse --is-inside-work-tree &>/dev/null && but claude post-tool || true",
            "timeout": 5
          }
        ]
      },
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/run-tests-async.sh",
            "timeout": 300,
            "statusMessage": "Running tests...",
            "async": true
          }
        ]
      },
      {
        "matcher": "Write|Edit|Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/observe-learning.sh post",
            "timeout": 5,
            "statusMessage": "Observing..."
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo '## Project Status' && (git status --short 2>/dev/null || echo 'Not a git repo') && echo '' && echo '## Recent TODOs' && (grep -r 'TODO:' src/ 2>/dev/null | head -5 || echo 'No TODOs found')",
            "timeout": 10
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if ~/.claude/hooks/guard-code-session.sh; then echo '{\"decision\": \"code session\"}'; else echo '{\"hookSpecificOutput\":{\"stopHookBehavior\":\"stop\"}}'; fi",
            "timeout": 5,
            "statusMessage": "Checking for code changes..."
          }
        ]
      },
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Review all code changes in this session for bugs, security issues, and quality problems. Report any findings.",
            "timeout": 120,
            "statusMessage": "Code quality review..."
          }
        ]
      },
      {
        "hooks": [
          {
            "type": "agent",
            "prompt": "Run git status and git diff --stat to check for uncommitted code changes. Report a summary of what changed (files modified, insertions, deletions). Do not run build, lint, or test commands.",
            "timeout": 60,
            "statusMessage": "Verifying tests..."
          }
        ]
      },
      {
        "hooks": [
          {
            "type": "command",
            "command": "but claude stop 2>/dev/null; terminal-notifier -message 'Claude Code finished' -title 'Claude Code' -sound default",
            "timeout": 10
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node ~/claude-memory/engine/memory-engine.mjs index --scope all 2>/dev/null || true",
            "timeout": 30,
            "statusMessage": "Flushing memory index..."
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "terminal-notifier -message 'Claude needs your input' -title 'Claude Code' -sound Basso",
            "timeout": 5
          }
        ]
      }
    ]
  },
  "alwaysThinkingEnabled": true
}
```

**Note:** The `but claude pre-tool` / `but claude post-tool` / `but claude stop` commands are from the [but.dev](https://but.dev) tool. Remove those hook entries if you don't use but.dev.

---

## 6. Global Instructions — ~/.claude/CLAUDE.md

File: `~/.claude/CLAUDE.md`

This is the primary behavioral configuration file. It contains:

- **Code Style** rules (minimal, production-ready, no TODOs/console.log)
- **ADL Protocol** (Stability > Explainability > Reusability > Scalability > Novelty)
- **Model Triage Protocol** (Opus orchestrates, Sonnet implements, Haiku searches)
- **Quality Gate system** (Gate 1 through Gate 3)
- **Effort Routing Protocol** (LOW/MEDIUM/HIGH effort per turn type)
- **Session Discipline** (one session = one PR cycle, 20-30 turn target)

```markdown
# Code Style

- Write minimal, production-ready code
- No comments explaining obvious code
- No debug statements, console.log, or TODOs
- No unnecessary abstractions or over-engineering
- Clear naming, no abbreviations
- Prefer clarity over cleverness
- Return early on errors
- Functions under 50 lines

# Decision Priority (ADL Protocol)

When choosing between approaches, apply this strict ordering:

**Stability > Explainability > Reusability > Scalability > Novelty**

Forbidden:
- Adding complexity to appear thorough or intelligent
- Making changes that can't be verified by the next gate
- Using vague concepts ("more robust", "cleaner") as justification — cite the specific failure it prevents
- Sacrificing a higher-priority dimension for a lower one

# Model Triage Protocol

Always act as a triage controller. Opus is the orchestrator — it plans, reviews, and decides. Delegate only work where lesser models reliably match Opus quality.

## Delegation Rules

**Delegate to Haiku** (only mechanical, zero-judgment tasks):
- File/code search and enumeration
- Reading files and returning raw content
- Simple text transformations with explicit specs

**Delegate to Sonnet** (clear-spec implementation only):
- Implementing functions/components from a detailed, unambiguous spec Opus wrote
- Writing tests when Opus specifies the exact cases
- Applying mechanical refactors (rename, extract, move) with explicit instructions
- Generating boilerplate/scaffolding from a template Opus defines

**Keep on Opus** (everything requiring judgment):
- All planning and architecture
- Writing specs and task decompositions
- Any task with ambiguous requirements
- Bug diagnosis and debugging
- Code review of all delegated work
- Security-sensitive code
- Integration logic and cross-file reasoning
- Final approval of all changes

## Context Window Escalation (API with 1M Context)

Default to Max plan (CLI subagents). Escalate to API 1M context only when: task can't be decomposed, 15+ tightly coupled files, decomposition was attempted and failed. API bridge: `~/claude-tools/api-bridge.mjs`. Usage log: `~/claude-tools/usage.log`.

## Codex CLI — Read-Only Subagent (OpenAI)

Read-only advisory contributor. Different model family = different blind spots. Output flows to Opus for judgment. Never touches codebase. Invoke for: code review, security review, debugging dead ends, architecture decisions, pre-PR sanity checks.

**Hard rules:** `--full-auto` + read-only always. Output to `/tmp/codex-*.md`. No style opinions. Opus is sole decision-maker.

## Gemini CLI — Read-Only Subagent (Google)

Same role as Codex, Google's model family. Invoke every time Codex is invoked. `-s -y` always. Output to `/tmp/gemini-*.md`. **CRITICAL**: `-s` flag does NOT prevent filesystem writes — always diff after Gemini runs.

**Command templates for all three tools:** Read `~/.claude/gate3-templates.md` when running Gate 3. Do not memorize the templates — read the file on demand.

## Quality Gate — Mandatory Review Protocol

**No code is considered complete until it passes all three gates.** This applies to all code — delegated to subagents, written by overnight batches, OR written directly by Opus in interactive sessions.

### Gate 1: Mechanical Checks (after every code change)

Run these before committing anything:

```bash
pnpm typecheck   # TypeScript strict mode
pnpm lint         # ESLint across all packages
pnpm build        # Next.js production build
```

If any gate fails, fix immediately. Do not commit broken code.

### Gate 1.5: Behavioral Verification (VBR — Verify Before Reporting)

After Gate 1 passes, before claiming a feature is complete:

1. **Verify outcome, not output.** "It compiles" is not "it works."
2. **Test the happy path.** Trace the user's click/request path.
3. **Test one adversarial path.** Ask: "What would a hostile or careless user do?"
4. **Only then report complete.**

### Gate 2: Code Review + Security Review (AUTO-ENFORCED)

A PreToolUse hook (`pre-commit-gate.sh`) blocks every `git commit` until both review agents have run. This is automatic — you cannot skip it.

**When the pre-commit gate blocks a commit:**

1. Run `git diff --cached` to identify staged changes
2. Dispatch **both agents in parallel** via the Task tool:
   - `code-reviewer` (subagent_type): "Review these staged changes for bugs, code quality, tenant isolation, audit completeness, and RBAC enforcement."
   - `security-reviewer` (subagent_type): "Review these staged changes for security vulnerabilities — OWASP Top 10, auth/authz gaps, injection, secrets exposure, tenant crosstalk."
3. Triage findings: classify each as VALID / FALSE POSITIVE
4. If issues found: fix them directly, re-stage, and re-run both agents
5. When both agents pass clean: run `touch /tmp/.claude-gate2-approved` then retry the commit
6. Only escalate to user if a finding is ambiguous or requires an architectural decision

**Override path:** If the user explicitly says "skip review" or the commit is trivial: run `touch /tmp/.claude-gate2-approved` without dispatching agents.

**Marker rules:** One-time-use (deleted after commit passes), expires after 30 minutes.

### Gate 2.5: Agent Memory Write-Back (after every Gate 2)

After evaluating Gate 2 findings, update agent memory:

1. **False positives?** — Append to `.claude/agent-memory/<agent>/calibration.md`
2. **New patterns?** — Append to `.claude/agent-memory/<agent>/patterns.md`
3. **Corrections?** — Append to the agent's `MEMORY.md` under "Recent Corrections"
4. **Codex/Gemini catches?** — Log in `calibration.md` with `[codex-catch]` or `[gemini-catch]`
5. **Re-index** — `node ~/claude-memory/engine/memory-engine.mjs index --scope agents`

### Gate 3: Cross-Model Review — Codex + Gemini (after every batch or feature)

Invoke both Codex (OpenAI) and Gemini (Google). Command templates: read `~/.claude/gate3-templates.md`.

**Triage protocol (MANDATORY):**

1. Read `/tmp/codex-review.md` and `/tmp/gemini-review.md`
2. Verify each finding against actual code
3. Classify: VALID / PARTIALLY VALID / FALSE POSITIVE
4. Tag: `[codex]`, `[gemini]`, or `[both]`
5. Present to user with implement/defer recommendation
6. User decides — Opus does not auto-implement Gate 3 findings

### When to Run Each Gate

| Scenario | Gate 1 | Gate 2 | Gate 3 |
|----------|--------|--------|--------|
| Subagent writes code | Yes | Yes | Yes |
| Overnight batch completes | Yes | Yes | Yes |
| Opus writes code interactively | Yes | Yes (if >3 files) | Yes (if security-sensitive) |
| Single-file lint/type fix | Yes | No | No |
| Build-error-resolver runs | Yes | Yes | No |

## Self-Improvement

After each triage session, record what worked and what didn't in:
`~/.claude/projects/-Users-$(whoami)/memory/model-triage.md`

# Effort Routing Protocol — Cost Control

Default effort is **medium** (set in env).

## LOW effort responses (be fast and minimal)
- Confirmatory, navigation, dispatch, mechanical commands, status checks

## MEDIUM effort responses (default — balanced)
- Standard implementation, configuration, troubleshooting, error diagnosis, Gate 2 dispatch

## HIGH effort responses (deep reasoning — use sparingly)
- Architecture/planning, Gate 2/3 triage, security review, bug diagnosis, spec writing

# Session Discipline — Cost Control

## One Session = One PR Cycle

Lifecycle: **plan → implement → Gate 1 → Gate 2 → Gate 3 → triage → fix → commit → clear.**

## Session Length Guardrails

- **Target**: 20-30 orchestrator turns per session
- **First compaction = wrap-up signal**: Finish current task, commit, write session notes, recommend clearing
- **Autocompact at 70%**: Prefer clearing over repeated compaction

## Context Lock Prevention

1. Delegate large reads to subagents
2. Never read full gate review outputs — use offset + limit
3. Use qmd for spec lookups
4. One compaction max per session
5. If locked: write state to `working-buffer.md`, tell user to clear
```

---

## 7. Project-Level Instructions — ~/CLAUDE.md

File: `~/CLAUDE.md`

This file configures the **persistent memory system**. Place it in `~/CLAUDE.md` so it applies when Claude Code runs from the home directory.

```markdown
# Claude Code Memory System

> Persistent memory at `~/claude-memory/`. Search engine at `~/claude-memory/engine/memory-engine.mjs`.

## Session Start Protocol

At the start of every session, **silently** (do not announce these steps):

1. Read `~/claude-memory/global/PROFILE.md` for user preferences
2. Read `~/claude-memory/global/KNOWLEDGE.md` for accumulated knowledge
3. Determine the current project from `~/claude-memory/config.json` or pwd
4. If `~/claude-memory/projects/<project>/CONTEXT.md` exists, read it
5. If `~/claude-memory/projects/<project>/DECISIONS.md` exists, read it
6. If today's session log exists, read it

## Memory Save Protocol

### What to Save Where
- **PROFILE.md** — User preferences, communication style, work patterns
- **KNOWLEDGE.md** — Technical facts, useful commands, gotchas, solutions
- **projects/<name>/CONTEXT.md** — Project architecture, stack, conventions
- **projects/<name>/DECISIONS.md** — Design decisions with rationale
- **projects/<name>/sessions/YYYY-MM-DD.md** — Session summaries

### WAL Protocol (Write-Ahead Logging)

Before composing a response, scan every user message for corrections, decisions, preferences, and specific values. If any are detected: **write to memory FIRST, then respond.**

### When to Save (Auto-Extraction Triggers)
- User states a preference
- A design decision is made
- A non-obvious technical fact is discovered
- User corrects an assumption
- A debugging session reveals a gotcha

## Memory Search Protocol

```bash
node ~/claude-memory/engine/memory-engine.mjs search "query text" --scope all
node ~/claude-memory/engine/memory-engine.mjs graph-query --entity "name"
```

## Working Buffer Protocol

At ~60% context usage, begin appending to `working-buffer.md`. After compaction, read buffer FIRST before recovery.

## Available Commands
- `/remember <info>` — Save to persistent memory
- `/recall <query>` — Search memory
- `/forget <query>` — Remove from search index
- `/memory-status` — Show system statistics
```

---

## 8. Hook Scripts

All hooks go in `~/.claude/hooks/`. Make each executable: `chmod +x ~/.claude/hooks/*.sh`

### 8a. is-code-context.sh — Shared Guard

Prevents coding hooks from firing during non-code sessions.

```bash
#!/bin/bash
# Shared guard: returns 0 if in a git repo, 1 otherwise.
# Usage: source ~/.claude/hooks/is-code-context.sh || exit 0

git rev-parse --is-inside-work-tree &>/dev/null 2>&1 || exit 1
```

### 8b. block-destructive.sh — Safety Guard

Blocks dangerous shell commands.

```bash
#!/bin/bash
# Hook: PreToolUse on Bash
# Blocks destructive shell commands before they execute

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$COMMAND" ]; then
  exit 0
fi

DESTRUCTIVE_PATTERNS=(
  "rm -rf /"
  "rm -rf ~"
  "rm -rf \."
  "rm -rf \*"
  "sudo rm -rf"
  "mkfs\."
  "dd if="
  "chmod -R 777 /"
  "chmod -R 777 ~"
  "> /dev/sd"
  ":(){ :|:&};:"
  "git clean -fdx"
  "git checkout \."
  "git restore \."
  "DROP TABLE"
  "DROP DATABASE"
  "TRUNCATE TABLE"
  "curl.*|.*sh"
  "curl.*|.*bash"
  "wget.*|.*sh"
  "wget.*|.*bash"
)

COMMAND_LOWER=$(echo "$COMMAND" | tr '[:upper:]' '[:lower:]')

for pattern in "${DESTRUCTIVE_PATTERNS[@]}"; do
  PATTERN_LOWER=$(echo "$pattern" | tr '[:upper:]' '[:lower:]')
  if echo "$COMMAND_LOWER" | grep -q "$PATTERN_LOWER"; then
    cat <<EOF
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"BLOCKED by safety hook: destructive command pattern detected ('$pattern'). If you need to run this, ask the user to run it manually."}}
EOF
    exit 0
  fi
done

exit 0
```

### 8c. block-patterns.sh — Code Quality Guard

Blocks TODO/FIXME comments and console.log in production code.

```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
CODE=$(echo "$INPUT" | jq -r '.tool_input.content // .tool_input.new_string // empty')

if [ -z "$CODE" ] || [ -z "$FILE" ]; then
  exit 0
fi

# Skip non-code files
case "$FILE" in
  *.json|*.md|*.yml|*.yaml|*.toml|*.ini|*.cfg|*.conf|*.sh|*.bash|*.lock)
    exit 0
    ;;
esac

# Block TODO/FIXME comments in source code
if echo "$CODE" | grep -qE '^\s*(//|#|/\*)\s*(TODO|FIXME|HACK|XXX):'; then
  echo "Blocked: Remove TODO/FIXME comments before committing" >&2
  exit 2
fi

# Block console.log in production code (skip test files)
if [[ "$FILE" != *".test."* && "$FILE" != *".spec."* && "$FILE" != *"__tests__"* ]]; then
  if echo "$CODE" | grep -qE '^\s*console\.(log|warn|error)\(' ; then
    echo "Blocked: Remove console statements from production code" >&2
    exit 2
  fi
fi

exit 0
```

### 8d. pre-commit-gate.sh — Auto Gate 2 Enforcement

Blocks `git commit` unless code-reviewer + security-reviewer agents have approved.

```bash
#!/bin/bash
# Hook: PreToolUse on Bash
# Blocks git commit unless Gate 2 review agents have approved
# Marker: /tmp/.claude-gate2-approved (one-time-use, 30-min expiry)

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$COMMAND" ]; then
  exit 0
fi

case "$COMMAND" in
  git\ commit*|"git commit"*)
    ;;
  *)
    exit 0
    ;;
esac

MARKER="/tmp/.claude-gate2-approved"

if [ -f "$MARKER" ]; then
  MARKER_AGE=$(( $(date +%s) - $(stat -f %m "$MARKER" 2>/dev/null || echo 0) ))
  if [ "$MARKER_AGE" -lt 1800 ]; then
    rm -f "$MARKER"
    exit 0
  fi
  rm -f "$MARKER"
fi

cat <<'EOF'
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"deny","permissionDecisionReason":"GATE 2 REQUIRED: Before committing, dispatch code-reviewer and security-reviewer agents on staged changes (git diff --cached). After both pass (or user overrides), run: touch /tmp/.claude-gate2-approved — then retry the commit."}}
EOF
exit 0
```

**Note:** `stat -f %m` is macOS-specific. On Linux, use `stat -c %Y`.

### 8e. observe-learning.sh — Continuous Learning (Plugin Wrapper)

```bash
#!/bin/bash
# Wrapper for continuous-learning-v2 observe hook
# Only runs in git repos (code contexts)

source ~/.claude/hooks/is-code-context.sh || exit 0

PHASE="${1:-pre}"
OBSERVE_SH=$(ls -d ~/.claude/plugins/cache/everything-claude-code/everything-claude-code/*/skills/continuous-learning-v2/hooks/observe.sh 2>/dev/null | tail -1)

if [ -n "$OBSERVE_SH" ] && [ -x "$OBSERVE_SH" ]; then
  exec "$OBSERVE_SH" "$PHASE"
fi
```

### 8f. run-tests-async.sh — Async Test Runner

```bash
#!/bin/bash
# Hook: PostToolUse on Write|Edit (async)
# Detects test framework and runs tests in background after file edits

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

if [ -z "$FILE_PATH" ]; then
  exit 0
fi

# Walk up to find project root
DIR=$(dirname "$FILE_PATH")
PROJECT_DIR=""
while [ "$DIR" != "/" ]; do
  if [ -f "$DIR/package.json" ] || [ -f "$DIR/pyproject.toml" ] || [ -f "$DIR/go.mod" ]; then
    PROJECT_DIR="$DIR"
    break
  fi
  DIR=$(dirname "$DIR")
done

if [ -z "$PROJECT_DIR" ]; then
  exit 0
fi

cd "$PROJECT_DIR" || exit 0

case "$FILE_PATH" in
  *.ts|*.tsx|*.js|*.jsx)
    if [ -f "package.json" ]; then
      if grep -q '"vitest"' package.json 2>/dev/null; then
        RESULT=$(npx vitest run --reporter=verbose 2>&1)
      elif grep -q '"jest"' package.json 2>/dev/null; then
        RESULT=$(npx jest --verbose 2>&1)
      elif grep -q '"test"' package.json 2>/dev/null; then
        RESULT=$(npm test 2>&1)
      else
        exit 0
      fi
      EXIT_CODE=$?
      if [ $EXIT_CODE -eq 0 ]; then
        echo "{\"systemMessage\": \"Tests PASSED after editing $(basename "$FILE_PATH")\"}"
      else
        RESULT=$(echo "$RESULT" | tail -30)
        echo "{\"systemMessage\": \"Tests FAILED after editing $(basename "$FILE_PATH"):\\n${RESULT}\"}"
      fi
    fi
    ;;
  *.py)
    if command -v python3 &>/dev/null; then
      if [ -f "pyproject.toml" ] || [ -f "pytest.ini" ] || [ -f "setup.cfg" ]; then
        RESULT=$(python3 -m pytest --tb=short 2>&1)
      else
        RESULT=$(python3 -m unittest discover 2>&1)
      fi
      EXIT_CODE=$?
      if [ $EXIT_CODE -eq 0 ]; then
        echo "{\"systemMessage\": \"Tests PASSED after editing $(basename "$FILE_PATH")\"}"
      else
        RESULT=$(echo "$RESULT" | tail -30)
        echo "{\"systemMessage\": \"Tests FAILED after editing $(basename "$FILE_PATH"):\\n${RESULT}\"}"
      fi
    fi
    ;;
  *.go)
    if command -v go &>/dev/null; then
      RESULT=$(go test ./... 2>&1)
      EXIT_CODE=$?
      if [ $EXIT_CODE -eq 0 ]; then
        echo "{\"systemMessage\": \"Tests PASSED after editing $(basename "$FILE_PATH")\"}"
      else
        RESULT=$(echo "$RESULT" | tail -30)
        echo "{\"systemMessage\": \"Tests FAILED after editing $(basename "$FILE_PATH"):\\n${RESULT}\"}"
      fi
    fi
    ;;
esac

exit 0
```

### 8g. guard-code-session.sh — Stop Hook Guard

Only runs Stop hooks (code review on exit) if in a git repo with changes.

```bash
#!/bin/bash
# Returns exit 0 (proceed) only if we're in a git repo with modified files

if ! git rev-parse --is-inside-work-tree &>/dev/null 2>&1; then
  exit 1
fi

CHANGES=$(git status --porcelain 2>/dev/null | head -1)
if [ -z "$CHANGES" ]; then
  exit 1
fi

exit 0
```

---

## 9. Custom Agents

All agents go in `~/.claude/agents/`. Each file defines an agent that can be dispatched via the Task tool.

### 9a. code-reviewer.md

```markdown
---
name: code-reviewer
description: >
  Read-only code reviewer that audits code for bugs, security issues, and
  maintainability problems. Use PROACTIVELY after writing or modifying code.
  MUST BE USED for all significant code changes before committing.
tools: Read, Grep, Glob
model: sonnet
memory: project
---

You are a senior staff engineer performing code review. You CANNOT edit files - you can only read and analyze.

## Memory Protocol

Before reviewing, read your memory files for project-specific knowledge:
- Your MEMORY.md contains known patterns, false positives, and calibration data
- After review, note any new patterns for the orchestrator to persist

## Review Protocol

1. Read memory for project-specific patterns and known false positives
2. Identify scope — find recently modified files or files specified by caller
3. Read each file fully — bugs hide in context
4. Analyze against checklist systematically
5. Cross-reference patterns from memory
6. Report findings in mandated format
7. Flag new patterns for memory persistence

## Checklist

### Critical (must fix)
- Security: injection, XSS, auth bypass, secrets in code
- Data loss: missing null checks, unhandled promise rejections
- Race conditions: shared mutable state, TOCTOU
- Logic errors: off-by-one, wrong operator, inverted condition
- Tenant isolation: missing brokerDealerId, RLS bypass

### Warning (should fix)
- Error handling: bare catch, swallowed errors
- Input validation: missing at API boundaries
- Performance: N+1 queries, unbounded loops, missing pagination
- Type safety: any casts, unsafe type assertions
- Audit completeness: mutations without auditLog()

### Suggestion (consider)
- Naming, complexity, duplication, dead code

## Output Format

## Code Review: [scope]
### Summary — [1-2 sentence assessment]
### Findings — CRITICAL/WARNING/SUGGESTION with file:line, issue, consequence, suggested fix
### New Patterns Discovered
### Verdict — [PASS | PASS WITH WARNINGS | NEEDS CHANGES]

## Principles

- Evidence over opinion — cite specific lines
- Severity matters — don't bury critical bugs under style nits
- Be direct
- No false positives — if unsure, say "verify whether..."
- Check memory first — don't re-flag known false positives
```

### 9b. security-reviewer.md

```markdown
---
name: security-reviewer
description: >
  Security vulnerability detection specialist. Use after writing code that
  handles authentication, user input, API endpoints, or sensitive data.
  Checks OWASP Top 10, FINRA compliance, tenant isolation, and crypto safety.
tools: Read, Grep, Glob
model: opus
memory: project
---

You are a security specialist reviewing code for vulnerabilities. You CANNOT edit files - you audit only.

## Memory Protocol

Before reviewing, read your memory files:
- MEMORY.md for known attack surfaces and threat model
- findings.md for past findings and resolution status

## Security Checklist

### Authentication & Authorization
- MFA enforcement on privileged roles
- withPermission() on every API route
- Session management: secure flags, proper expiry

### Tenant Isolation (CRITICAL)
- brokerDealerId in every tenant-scoped query
- No raw SQL bypassing RLS
- Cross-tenant data leak in any response

### Input Handling
- Zod validation on all API inputs
- request.json() wrapped in try/catch
- No SQL injection or XSS

### Cryptography & Secrets
- HMAC-SHA256+ for tokens, timing-safe comparison
- No secrets in logs, errors, or responses

### Rate Limiting & DoS
- Rate limits on public endpoints
- Bounded data structures, pagination

### Audit & Compliance
- auditLog() on every state mutation
- Hash chain integrity, WORM compliance

## Output Format

## Security Review: [scope]
### Threat Model — attackers, targets, attack surface
### Findings — CRITICAL/HIGH/MEDIUM/LOW with file:line, attack vector, impact, remediation, CVSS
### Threat Model Updates
### Verdict — [SECURE | NEEDS REMEDIATION]
```

### 9c. planner.md

```markdown
---
name: planner
description: >
  Software architect agent for designing implementation plans. Use when
  planning new features, decomposing complex tasks, or making architectural
  decisions. Returns step-by-step plans with file lists and risk assessment.
tools: Read, Grep, Glob
model: opus
memory: project
---

You are an expert software architect creating implementation plans. You CANNOT edit files - you research and plan only.

## Memory Protocol

Read MEMORY.md for estimation calibration data and decomposition patterns.

## Decomposition Rules

- Each subtask completable by a single agent in one session
- Independent subtasks should be parallelizable
- Shared-file subtasks must be sequenced
- Security-sensitive subtasks stay on Opus
- Mechanical subtasks with exact specs go to Sonnet

## Output Format

## Implementation Plan: [title]
### Scope
### Risk Assessment — complexity, security sensitivity, cross-cutting concerns
### Prerequisites
### Subtasks — numbered, with model assignment, files, dependencies, spec, estimated complexity
### Estimation Record — file count, subtask count, parallelizable groups
```

### 9d. build-error-resolver.md

```markdown
---
name: build-error-resolver
description: >
  Build and TypeScript error resolution specialist. Use when build fails or
  type errors occur. Fixes build/type errors only with minimal diffs, no
  architectural edits. Focuses on getting the build green quickly.
tools: Read, Write, Edit, Bash, Grep, Glob
model: sonnet
memory: project
---

You are a build error resolution specialist. Your ONLY job is to fix build, typecheck, and lint errors with minimal changes.

## Memory Protocol

Read MEMORY.md for known error patterns and proven fixes.

## Resolution Protocol

1. Run the failing command
2. Parse errors — file, line, error code, message
3. Check memory for known fixes
4. Apply minimal fix — smallest change that resolves the error
5. Re-run to verify
6. Report what was fixed

## Hard Rules

- NEVER use `any` to fix a type error
- NEVER use `@ts-ignore` or `@ts-expect-error`
- NEVER use `as` casts except on Zod parse boundaries
- NEVER delete functionality to fix a build error
- Keep changes under 5 lines per file when possible
- If a fix requires >20 lines, escalate to the orchestrator
```

### 9e. code-quality.md

```markdown
---
name: code-quality
description: Verify code quality and production readiness
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer focused on:

1. **Minimalism**: No unnecessary abstractions
2. **Clarity**: Obvious names and structure
3. **Testing**: Comprehensive, high coverage
4. **Standards**: Language conventions
5. **Production-Ready**: Error handling, edge cases
6. **Linting**: Passes all checks
7. **Performance**: No obvious inefficiencies

Verify the implementation is production-ready.
```

---

## 10. Plugins

Plugins are installed via the Claude Code CLI. Install each:

```bash
claude plugin install frontend-design@claude-plugins-official
claude plugin install context7@claude-plugins-official
claude plugin install ralph-loop@claude-plugins-official
claude plugin install code-review@claude-plugins-official
claude plugin install playwright@claude-plugins-official
claude plugin install security-guidance@claude-plugins-official
claude plugin install superpowers@claude-plugins-official
claude plugin install code-simplifier@claude-plugins-official
claude plugin install everything-claude-code@everything-claude-code
```

### What Each Plugin Provides

| Plugin | Purpose |
|--------|---------|
| **superpowers** | Brainstorming, plan writing, TDD, debugging, git worktrees, verification, code review workflows |
| **everything-claude-code** | Comprehensive agent toolkit: TDD, continuous learning, coding standards, security review, strategic compaction, QMD MCP server |
| **context7** | MCP server for querying up-to-date library documentation |
| **playwright** | Browser automation MCP — screenshot, click, navigate, fill forms, evaluate JS |
| **frontend-design** | Production-grade UI/UX component generation |
| **code-review** | Code review workflow skills |
| **security-guidance** | Security-focused review skills |
| **ralph-loop** | Iterative refinement loop |
| **code-simplifier** | Code clarity and maintainability refactoring |

---

## 11. Memory System

### 11a. Directory Structure

```
~/claude-memory/
├── config.json              # Project mappings and search config
├── global/
│   ├── PROFILE.md           # User preferences and identity
│   ├── KNOWLEDGE.md         # Accumulated technical knowledge
│   └── memory.db            # SQLite search index (auto-generated)
├── engine/
│   └── memory-engine.mjs    # Search engine script
└── projects/
    └── <project-name>/
        ├── CONTEXT.md        # Project architecture
        ├── DECISIONS.md      # Design decisions
        └── sessions/
            └── YYYY-MM-DD.md # Daily session logs
```

### 11b. config.json

File: `~/claude-memory/config.json`

```json
{
  "search": {
    "topK": 10,
    "ftsWeight": 0.4,
    "vectorWeight": 0.6,
    "rrf_k": 60
  },
  "user": {
    "name": "<YOUR_NAME>",
    "email": "<YOUR_EMAIL>"
  },
  "projects": {
    "<project-slug>": {
      "path": "/path/to/project",
      "description": "Project description"
    }
  },
  "agentMemory": {
    "dirs": [
      "/path/to/project/.claude/agent-memory"
    ],
    "agents": ["code-reviewer", "build-error-resolver", "planner", "security-reviewer"]
  }
}
```

### 11c. Memory Engine

The memory engine (`~/claude-memory/engine/memory-engine.mjs`) is a Node.js script that provides:

- **FTS5 full-text search** over markdown files
- **Vector search** via local embeddings (embeddinggemma-300M-Q8_0)
- **Graph database** for entity relationships
- **RRF fusion** combining FTS5 + vector + graph scores

Commands:
```bash
# Index all memory files
node ~/claude-memory/engine/memory-engine.mjs index --scope all

# Search across all scopes
node ~/claude-memory/engine/memory-engine.mjs search "query" --scope all

# Graph: add relationship
node ~/claude-memory/engine/memory-engine.mjs graph-add "source" "relation" "target" --source-type concept --target-type technology

# Graph: query entity
node ~/claude-memory/engine/memory-engine.mjs graph-query --entity "name"
```

### 11d. Agent Memory

For projects with persistent agent memory:

```
<project>/.claude/agent-memory/
├── shared/
│   └── MEMORY.md
├── code-reviewer/
│   ├── MEMORY.md
│   ├── calibration.md    # False positive tracking
│   └── patterns.md       # Recurring issue patterns
├── security-reviewer/
│   ├── MEMORY.md
│   └── findings.md
├── planner/
│   ├── MEMORY.md
│   └── estimation.md
└── build-error-resolver/
    ├── MEMORY.md
    └── solutions.md
```

### 11e. Initial PROFILE.md

Create `~/claude-memory/global/PROFILE.md`:

```markdown
# User Profile

## Identity
- **Name**: <YOUR_NAME>
- **Email**: <YOUR_EMAIL>

## Preferences
- **Code Style**: Minimal, production-ready, no over-engineering
- **AI Tools**: Claude Code CLI as primary development environment
- **Quality > Economy**: Never skip memory write-back or review steps to save tokens

## Work Patterns
- **Projects**: <list active projects>

## Tools & Environment
- **OS**: macOS
- **Node.js**: <version>
- **Shell**: zsh
```

---

## 12. Gate 3 Templates

File: `~/.claude/gate3-templates.md`

```markdown
# Gate 3 Command Templates

Reference file for Codex CLI and Gemini CLI invocation patterns.

## Context Efficiency Rules
- Output to files: `/tmp/codex-review.md` or `/tmp/gemini-review.md`
- Read summaries only after running — skip preamble
- Large PRs (>8 services): split into 2-3 runs by domain
- File piping: use `git diff --name-only`, pipe contents via `cat`

## Codex CLI Invocation

### Spec-anchored review (preferred)
```bash
(
  echo "Can you review the following code? Look for code items for review. Not just code correctness, but function checking code after compile, with full context. Max effort. Make sure [SPEC_NAME] is followed."
  echo ""
  echo "=== SPEC_FILE ==="
  cat "path/to/authoritative-spec.md"
  echo ""
  for f in $(git diff --name-only HEAD~1 HEAD -- '*.ts' '*.tsx'); do
    echo "=== $f ==="; cat "$f"
  done
) | codex exec -s read-only --full-auto -o /tmp/codex-review.md -
```

### Standard review (no spec)
```bash
(
  echo "Review this code for bugs, security issues, and edge cases. FINRA compliance context: check tenant isolation, audit logging, RBAC enforcement, TypeScript strictness. Cite filenames and line numbers. Only flag real issues."
  echo ""
  for f in $(git diff --name-only HEAD~1 HEAD -- '*.ts' '*.tsx'); do
    echo "=== $f ==="; cat "$f"
  done
) | codex exec -s read-only --full-auto -o /tmp/codex-review.md -
```

## Gemini CLI Invocation

### Code review (multi-file)
```bash
(
  echo "Review this code for bugs, security issues, and edge cases. FINRA compliance context. Cite filenames and line numbers. Only flag real issues, not style."
  echo ""
  for f in $(git diff --name-only HEAD~1 HEAD -- '*.ts' '*.tsx'); do
    echo "=== $f ==="; cat "$f"
  done
) | gemini -s -y -p "You are a read-only code reviewer. Analyze the code provided via stdin." > /tmp/gemini-review.md 2>&1
```

## API Bridge Invocation

```bash
# Standard context (200k)
node ~/claude-tools/api-bridge.mjs --model sonnet --task "description" --files file1.ts

# 1M context with glob
node ~/claude-tools/api-bridge.mjs --model sonnet --context-1m --task "description" --glob "src/**/*.ts"
```
```

---

## 13. Custom Tools — ~/claude-tools/

### 13a. api-bridge.mjs

A CLI wrapper for delegating to Claude models (Opus/Sonnet/Haiku) with streaming, cost tracking, and 1M context support.

```bash
# Create the directory and install dependencies
mkdir -p ~/claude-tools
cd ~/claude-tools
npm init -y
npm install @anthropic-ai/sdk

# Store your API key in macOS Keychain (not in files)
security add-generic-password -a "anthropic-api" -s "anthropic-api-key" -w "<YOUR_API_KEY>"
```

The `api-bridge.mjs` script (~333 lines) provides:
- `--model opus|sonnet|haiku` — select model
- `--task "description"` — the prompt
- `--files file1.ts file2.ts` — include file contents
- `--glob "src/**/*.ts"` — glob for files
- `--context-1m` — enable 1M context window
- `--system "prompt"` — custom system prompt
- `--output /path/to/file.md` — save output to file
- Cost tracking in `~/claude-tools/usage.log` (TSV format)

### 13b. notify.sh

iMessage notification script for alerting when Claude needs input or finishes work.

```bash
#!/bin/bash
# Usage: ~/claude-tools/notify.sh "message" [info|action|blocked]
MESSAGE="${1:-Claude Code notification}"
TYPE="${2:-info}"
PHONE="<YOUR_PHONE_NUMBER>"

osascript -e "tell application \"Messages\" to send \"[$TYPE] $MESSAGE\" to buddy \"$PHONE\" of (1st account whose service type = iMessage)"
```

### 13c. agent-memory-flush.sh

Post-agent memory consolidation script (for SubagentStop hooks in project settings).

```bash
#!/bin/bash
# Flushes agent memory after subagent completes
node ~/claude-memory/engine/memory-engine.mjs index --scope agents 2>/dev/null || true
```

---

## 14. QMD Local Search Engine

QMD indexes local markdown documents for fast search via MCP.

```bash
# Install
bun install -g qmd

# Initialize in a docs directory
cd /path/to/your/docs
qmd init

# Index and embed
qmd update
qmd embed

# After editing any docs, re-index:
qmd update && qmd embed
```

QMD provides these MCP tools to Claude Code:
- `mcp__qmd__search` — keyword search (~30ms)
- `mcp__qmd__vector_search` — semantic search (~2s)
- `mcp__qmd__deep_search` — auto-expanded multi-strategy search (~10s)
- `mcp__qmd__get` — retrieve full document
- `mcp__qmd__multi_get` — batch retrieve by glob
- `mcp__qmd__status` — index health check

---

## 15. Auto Memory — Project Memory

Claude Code's built-in auto-memory at `~/.claude/projects/-Users-<username>/memory/MEMORY.md`.

This file is automatically loaded into every session's system prompt. Use it for:
- Project-specific knowledge and gotchas
- Critical rules distilled from past sessions
- Phase/status tracking
- Known Sonnet delegation failure patterns
- Security fixes and their resolution status

Keep it under 200 lines (content after line 200 is truncated from the system prompt).

---

## 16. Verification Checklist

After setting up, verify everything works:

```bash
# 1. Claude Code runs
claude --version

# 2. Settings loaded
claude config list  # or check ~/.claude/settings.json

# 3. Hooks are executable
ls -la ~/.claude/hooks/*.sh

# 4. Agents exist
ls ~/.claude/agents/

# 5. Memory system works
node ~/claude-memory/engine/memory-engine.mjs search "test" --scope all

# 6. Plugins installed
# Open Claude Code and check available skills

# 7. Cross-model review tools
codex --version
gemini --version

# 8. QMD works (if docs indexed)
# In Claude Code, use mcp__qmd__status

# 9. Hooks fire correctly
# In a git repo: edit a file, verify prettier runs
# Try `git commit` without Gate 2 marker — should be blocked

# 10. Notifications work
terminal-notifier -message "Test notification" -title "Claude Code"
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    Claude Code CLI                       │
│                  (Opus orchestrator)                      │
├─────────────┬─────────────┬─────────────┬───────────────┤
│  PreToolUse │ PostToolUse │    Stop     │  PreCompact   │
│   Hooks     │   Hooks     │   Hooks    │    Hooks      │
├─────────────┼─────────────┼────────────┼───────────────┤
│ block-      │ prettier    │ guard-code │ memory-       │
│ destructive │ observe-    │ code-review│ engine index  │
│ block-      │ learning    │ diff-stat  │               │
│ patterns    │ run-tests   │ notify     │               │
│ pre-commit  │             │            │               │
│ gate        │             │            │               │
│ observe-    │             │            │               │
│ learning    │             │            │               │
├─────────────┴─────────────┴────────────┴───────────────┤
│                    Task Tool (Subagents)                 │
├──────────┬──────────┬──────────┬──────────┬────────────┤
│ code-    │ security │ planner  │ build-   │ code-      │
│ reviewer │ reviewer │ (Opus)   │ error-   │ quality    │
│ (Sonnet) │ (Opus)   │          │ resolver │ (Opus)     │
│          │          │          │ (Sonnet) │            │
├──────────┴──────────┴──────────┴──────────┴────────────┤
│                    Gate 3 (Cross-Model)                  │
├────────────────────────┬────────────────────────────────┤
│   Codex CLI (OpenAI)   │     Gemini CLI (Google)        │
│   read-only, --full-auto│    read-only, -s -y            │
│   → /tmp/codex-*.md    │    → /tmp/gemini-*.md          │
├────────────────────────┴────────────────────────────────┤
│                    Memory System                         │
├──────────┬──────────┬──────────┬────────────────────────┤
│ PROFILE  │ KNOWLEDGE│ Project  │  Agent Memory           │
│ .md      │ .md      │ CONTEXT  │  calibration.md         │
│          │          │ DECISIONS│  patterns.md             │
│          │          │ sessions/│  solutions.md            │
├──────────┴──────────┴──────────┴────────────────────────┤
│                    MCP Servers                            │
├──────────────┬──────────────┬────────────────────────────┤
│  Context7    │  Playwright  │  QMD (local docs search)   │
│  (lib docs)  │  (browser)   │  (Code Plan/Project Brief) │
└──────────────┴──────────────┴────────────────────────────┘
```

---

## Notes

- **Secrets**: Store API keys in macOS Keychain (`security add-generic-password`), never in files. The api-bridge.mjs reads from Keychain at runtime.
- **Gemini sandbox bug**: The `-s` flag does NOT prevent Gemini from writing files. Always diff after Gemini runs or use a read-only git worktree.
- **macOS-specific**: `stat -f %m` (epoch seconds), `terminal-notifier`, `osascript` for iMessage. Adapt for Linux.
- **Plugin cache**: Plugins install to `~/.claude/plugins/cache/`. They update automatically.
- **but.dev hooks**: The `but claude pre-tool` / `post-tool` / `stop` commands are from but.dev. Remove those hook entries if not using but.dev.

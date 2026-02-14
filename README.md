# Claude Code Configuration

A comprehensive setup document for replicating a production-grade Claude Code CLI environment from scratch.

## What's Included

- **System prerequisites** — Node.js, pnpm, jq, Codex CLI, Gemini CLI, qmd, and more
- **Settings files** — `settings.json` (base) and `settings.local.json` (session overrides with hooks)
- **CLAUDE.md instructions** — Quality gates, effort routing, session discipline, model triage
- **7 hook scripts** — Safety guards, code quality blocks, auto Gate 2 enforcement, async test runner
- **5 custom agents** — code-reviewer, security-reviewer, planner, build-error-resolver, code-quality
- **9 plugins** — superpowers, everything-claude-code, Context7, Playwright, and more
- **Memory system** — SQLite FTS5 + vector search + knowledge graph with persistent agent memory
- **Gate 3 templates** — Codex (OpenAI) and Gemini (Google) cross-model review invocation patterns
- **Custom tools** — API bridge for 1M context delegation, iMessage notifications

## Quick Start

1. Install system prerequisites (Section 1)
2. Install Claude Code (Section 2)
3. Create directory structure (Section 3)
4. Copy settings files (Sections 4-5)
5. Write CLAUDE.md instructions (Sections 6-7)
6. Create hook scripts and `chmod +x` them (Section 8)
7. Create agent definitions (Section 9)
8. Install plugins (Section 10)
9. Set up memory system (Section 11)
10. Run verification checklist (Section 16)

## Architecture

```
Claude Code CLI (Opus orchestrator)
├── PreToolUse Hooks (safety, quality, Gate 2 enforcement)
├── PostToolUse Hooks (prettier, tests, learning)
├── Stop Hooks (code review on exit, notifications)
├── Task Agents (code-reviewer, security-reviewer, planner, build-error-resolver)
├── Gate 3 (Codex CLI + Gemini CLI cross-model review)
├── Memory System (FTS5 + vector + graph, per-project + per-agent)
└── MCP Servers (Context7, Playwright, QMD)
```

## Security

All secrets, API keys, and PII are replaced with `<REDACTED>` or `<YOUR_*>` placeholders. Store credentials in macOS Keychain, never in plaintext files.

## License

MIT

---
title: OpenCode V2 Setup & Token Discipline
tags:
  - opencode
  - ai
  - agentic-coder
  - workflow
  - documentation
  - system-spec
---

# 🧠 OpenCode V2 Setup & Token Discipline

> **What this is:** Reference documentation for the OpenCode V2 coding-agent installation on this machine — configuration, model economy, skill arsenal, and the token-conservation strategy that keeps free-tier usage sustainable.

---

## 1. What OpenCode Is

OpenCode is a **client-server coding agent**: the TUI (client) talks to a shared background service that owns sessions, configuration, permissions, and tool execution. Docs live at `https://opencode.ai/v2/docs/`. It is distinct from Claude Code / Antigravity patterns captured in [[You are using AI the wrong way (Jaydee Ballajo)]] and [[Agentic System Architecture & Skill Verification]], but follows the same boss/worker philosophy.

## 2. Configuration Layout

| File | Path | Role |
| --- | --- | --- |
| Global config | `~/.config/opencode/opencode.jsonc` | Model defaults, agent models, references, permissions |
| CLI/TUI prefs | `~/.config/opencode/cli.json` | Themes, keybinds, terminal behavior |
| Global instructions | `~/.config/opencode/AGENTS.md` | Operating rules applied to every session |
| Skills | `~/.config/opencode/skills/<name>/SKILL.md` | Inert Markdown behavior modules, loaded on demand |
| Plugins | `~/.config/opencode/plugins/graphify.js` | Vault graph indexing hook |
| Service | `opencode service status / restart` | Background daemon lifecycle |

Config is **hot-reloaded** — no restart needed after edits. Project-level `opencode.json(c)` merges over global.

## 3. Model Economy (as configured)

| Role | Model | Why |
| --- | --- | --- |
| Session default | `opencode/glm-5.3-flash-free` | Free, zero-retention, no training on data |
| `explore` / `title` / `summary` agents | `opencode/nemotron-3.5-lightning-free` | Cheap, fast, keeps heavy read work off the paid tier |
| Fallback ladder | other free models → `claude-haiku-5.5` / `gpt-5-nano` | Rotation when a quota is spent; pennies per turn paid |

**The two limits that matter:**

1. **Context window** — every turn retransmits the *entire* conversation + system prompt + tool schemas, so session cost compounds ~O(n²). Auto-compaction fires only at ~85% full and is lossy (keeps a 15k-token recent tail + summary).
2. **Free-model quota** — enforced server-side, undocumented, rolling. A spent model simply errors until the window resets. Rotation is the counter.

## 4. Skill Arsenal

### Inherited (12)
`code-vault-bridge`, `git-sentinel`, `graphify`, `graphify-navigator`, `hallmark`, `karpathy-verifier`, `obsidian-vault-manager`, `obsidian-vault-refactor`, `ponytail`, `self-discovery-analyst`, `test-driven-enforcer`, `ui-ux-pro-max`

### Token-Defense Pack (6, added Nov 2026)

| Skill | Counters | Core rule |
| --- | --- | --- |
| `token-discipline` | Greedy tool output | Grep-first, bounded reads, distill-then-return |
| `delegate-or-die` | Exploration polluting main session | >2 files → subagent with output contract |
| `session-hygiene` | Multi-topic session bloat | Pivot test, handoff blocks, one task = one session |
| `three-strikes` | Runaway retry loops | Max 2 fix attempts per failure, then report |
| `checkpoint` | Late, lossy compaction | Manual compaction at milestones, not at 85% |
| `budget-watch` | Quota/context blindspot | 50%/75% context gauges, model rotation ladder |

Skills chain: `budget-watch` detects → `token-discipline` + `delegate-or-die` stop the leak → `session-hygiene` + `checkpoint` manage the remainder → `three-strikes` caps the spin. They are *incentives*, not hard gates — enforcement-grade behavior would need a plugin hook.

## 5. Dual-Path Workspace (see also global AGENTS.md)

- **Knowledge Base** (this vault): documentation only — never code or build artifacts.
- **Codebase** (`C:\Users\ASUS\Documents (ASUS)\PROGRAMMING`): all projects and scripts.
- Both are wired as `references` in `opencode.jsonc`; vault reads are allowed, vault **edits require asking**, git mutations always ask.

## 6. Related

* [[You are using AI the wrong way (Jaydee Ballajo)]] — the CLI/subagent philosophy this setup implements.
* [[Agentic System Architecture & Skill Verification]] — skill architecture benchmark this setup parallels.
* [[Codebase Understanding & Summary Generation Prompt]] — companion onboarding prompt for agentic tools.
* [[Personal Knowledge Hub (Map of Content)|Personal Knowledge Hub]] — parent MOC.

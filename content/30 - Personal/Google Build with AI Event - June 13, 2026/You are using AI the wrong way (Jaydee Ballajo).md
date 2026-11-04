---
title: You are using AI the wrong way (Jaydee Ballajo)
tags:
  - personal
  - ai-event
  - google-ai
  - cli-agents
  - context-engineering
  - subagents
  - ai-security
---

# 💻 You Are Using AI the Wrong Way — CLI Workflows & Subagent Systems

## 🎯 Keynote: Jaydee Ballajo

## 🎯 Stop Using AI Exclusively from the Browser
- Treating AI solely like a search engine or chat window underutilizes its autonomous and agentic potential.

## 🔍 Gemini CLI & Context Management
- In **CLI environments** (Gemini CLI / Claude Code), you have granular visibility into token consumption.
- **`.md` Files as Context:** Local Markdown files act as persistent project memory.
- Progressive disclosure and localized context files eliminate the need for bloated conversational threads.

## 🔍 Subagent Architecture & Design
- **Progressive Disclosure:** Structure documentation so agents only load detailed specifications when required.
- **Description & Triggers:** Agent routing depends on sharp tool/skill descriptions and clear trigger conditions.
- **"Borrow, Don't Build":** Leverage open-source specialist subagents available across GitHub.
- **Hierarchical Tiering ("Smart Boss / Cheap Workers"):**
  - **Smart Boss (e.g. Claude 3.5/3.7 Sonnet / Opus / Gemini Pro):** Orchestration, high-level planning, synthesis.
  - **Cheap Workers (e.g. Haiku / Flash):** Bulk file reading, syntax checking, low-latency execution.
- **Rule of Thumb for Subagents:** Delegate tasks that require reading many files to keep primary context clean.

## 🛡️ AI Security in the CLI
- Is it safe to execute AI agents in terminal environments?
- **Security Checklist:**
  - Run agent environments inside **containers** (Docker, WSL sandbox).
  - **Never grant root / sudo access** without oversight.
  - Review destructive commands before approval.

## ❓ Learning vs. Offloading
- Would you consider an AI to be a teacher?
- Don't let the AI simply do the assignment for you—use it to deepen understanding and accelerate mastery.

---

## 🔗 Connections & Related Notes
* **Event Hub:** [[Google Build with AI Event — June 13, 2026 (Hub)|Google Build with AI Event]]
* **Agentic Automation:** [[Agentic Workflow The Future of Intelligent Automation - Anti Gravity (Larry Mark Somocor)]] — Autonomous workflows, slash commands, and multi-agent coordination.
* **Onboarding Codebase Agent:** [[Codebase Understanding & Summary Generation Prompt]] — Practical application of `.md` context injection for AI coders.
* **Rapid Prototyping:** [[Build client ready website fast with Claude (Jlo Edios)]] — Context routers and `.claude.md` skills.
* **Context Optimization:** [[GENERAL TAKE AWAY TIPS]] — Utilizing lean markdown files for token efficiency.
* **Security & Inspection:** [[Incident or Building Inspection]] — Verifying integrity and sandboxing environments.

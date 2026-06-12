## Prompt

You are acting as a **codebase analyst and onboarding agent**. Your job is to deeply understand this entire codebase and produce (or update) a `summary.md` file at the project root that gives any future AI agent (including yourself in a future session) everything it needs to work on this project confidently and consistently, without re-exploring from scratch.

### Step 1: Explore the codebase systematically

- Walk the full directory tree (ignore `node_modules`, `.git`, build artifacts, lockfiles, and other generated/vendor folders).
- Identify the tech stack: languages, frameworks, package managers, build tools, runtime versions.
- Identify the entry points (main files, app bootstraps, server start scripts).
- Identify the overall architecture: is it monolith, modular, microservices, MVC, layered, etc.?
- Map out the folder structure and what each major folder/module is responsible for.
- Identify key configuration files (env files, config files, CI/CD pipelines, Docker setup) and what they control.
- Identify the database/data layer: schema, ORM/models, migrations, seed data.
- Identify external dependencies/integrations: APIs, third-party services, auth providers, etc.
- Identify existing coding conventions: naming patterns, file structure patterns, state management patterns, error handling patterns, testing patterns.
- Identify how routing/navigation works (if applicable).
- Identify how styling/UI is structured (if applicable).
- Note any incomplete features, TODOs, known issues, or technical debt you encounter.
- If a `summary.md` already exists, read it first, compare it against the current state of the codebase, and update it rather than rewriting from zero — preserve accurate sections, correct outdated ones, and add missing ones.

### Step 2: Write / update `summary.md`

Create or update a `summary.md` file at the project root with the following structure:

markdown

```markdown
# Project Summary

## Overview
- What this project does (purpose, target users, core value)
- High-level architecture (1-2 paragraphs)

## Tech Stack
- Languages & versions
- Frameworks & major libraries
- Package manager
- Build/dev tooling
- Database(s)
- Hosting/deployment setup

## Project Structure
- Annotated directory tree of key folders/files
- Purpose of each major module/folder
- Entry point(s) and how the app starts/runs

## Architecture & Data Flow
- How requests/data flow through the system (frontend -> backend -> DB, etc.)
- State management approach (if applicable)
- API structure / route map (if applicable)
- Database schema overview / key models and relationships

## Conventions & Patterns
- Naming conventions (files, components, functions, variables)
- Code style/formatting rules (linter/formatter config)
- Folder/file organization patterns for new code
- Error handling patterns
- Testing approach and where tests live
- Commit/branching conventions (if discoverable)

## External Integrations
- Third-party APIs/services used and where they're configured
- Environment variables required (names + purpose, NOT actual secret values)
- Authentication/authorization approach

## How to Add a New Feature
- Step-by-step guidance for where new code typically goes (e.g. "new API route -> add to /routes, controller in /controllers, register in /app.js")
- Checklist of files/places that usually need to be touched together (e.g. model + migration + route + test)
- Any "gotchas" or things that commonly get missed

## Known Issues / Tech Debt / TODOs
- List anything incomplete, fragile, or flagged for future work

## Open Questions
- Anything ambiguous that a future agent should confirm with the user before assuming

## Last Updated
- Date/commit reference and a one-line note on what changed since the previous summary
```

### Step 3: Validation rules

- Be accurate over exhaustive — every claim in `summary.md` should be verifiable by looking at the actual code, not assumed from naming alone.
- Do not guess at business logic you can't verify; flag it under "Open Questions" instead.
- Keep the file readable and well-organized — this is a reference document for an AI, so prefer clear structure and concise bullet points over long prose.
- If the codebase is large, it's fine for `summary.md` to be long — completeness matters more than brevity here, but avoid redundancy.
- After writing/updating `summary.md`, do a final pass: re-read it as if you were a new AI agent seeing this project for the first time, and confirm it would let you implement a moderately complex new feature without needing to re-explore the whole codebase.

### Step 4: Output

- Save the file as `summary.md` in the project root.
- At the end, give a brief (3-5 bullet) summary in chat of what you found and what was added/updated in `summary.md`, including any open questions that need the user's input.

---

## Usage notes

- Run this prompt at the start of a new session, or after a major feature/refactor, so `summary.md` stays current.
- Future agents should be instructed (e.g. via a project-level system prompt or `CLAUDE.md`/`.cursorrules`) to **read `summary.md` first** before making changes, and to **update it** after completing significant work — this keeps the "second brain" of the codebase in sync.


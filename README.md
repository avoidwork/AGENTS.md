# AGENTS.md

A safe, starter `AGENTS.md` for your project. It bootstraps a deterministic workflow
that you and your agents can easily extend.

---

## What is this?

`AGENTS.md` is a convention file placed at the root of a project that defines rules,
conventions, and session learnings for AI agents working on the codebase. Think of it as
the agent equivalent of `CONTRIBUTING.md` — it tells agents how to behave, what patterns
are forbidden, and how the project is structured.

This starter file provides:

- **Security guards** — forbidden patterns (no `eval`, no hardcoded secrets, OWASP
  Top 10 awareness).
- **Coding standards** — Python style, FastAPI conventions, LangGraph patterns.
- **Git workflow** — conventional commits, no force-push, no rebase without agreement.
- **Operational rules** — 100% coverage, pre-commit hooks, mocking guidelines.

## How to extend

Edit `AGENTS.md` directly. Add sections, tighten rules, or bake in project-specific
conventions. Agents reading the file will follow whatever is written there.

## Structure

| Section | Purpose |
|---------|---------|
| `1. Core Rules` | Forbidden patterns, security mandates, core principles (DRY, KISS, YAGNI) |
| `2. Project Context` | Description, layout, quick commands |
| `3. Python Conventions` | Tooling, style, error handling, async, testing |
| `4. Framework Conventions` | FastAPI, LangGraph, auth modes |
| `5. Git Conventions` | Commits, branching, review |
| `6. Operational Rules` | Coverage, pre-commit, mocking gotchas |
| `7. Session Learnings` | Codebase-specific discoveries |
| `8. Checklist` | Pre-markdown criteria |

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
- **Coding standards** — an example of Python style, FastAPI conventions, LangGraph patterns. Replace with your language/toolchain.
- **Git workflow** — conventional commits, no force-push, no rebase without agreement.
- **Operational rules** — 100% coverage, pre-commit hooks, mocking guidelines.

## How to extend

Edit `AGENTS.md` directly. Add sections, tighten rules, or bake in project-specific
conventions. Agents reading the file will follow whatever is written there.

Replace the language-specific examples (sections 1, 3–4, 6, 8) with the conventions for
your project's stack. Keep the security rules, git conventions, and operational rules as a
solid default — they apply regardless of language.

## Structure

| Section | Purpose |
|---------|---------|
| `1. Core Rules` | Forbidden patterns, security mandates, core principles (DRY, KISS, YAGNI) |
| `2. Project Context` | Description, layout, quick commands |
| `3. Python Conventions` | Example — tooling, style, error handling, async, testing (adapt for your language) |
| `4. Framework Conventions` | Example — FastAPI, LangGraph, auth modes (adapt for your frameworks) |
| `5. Git Conventions` | Commits, branching, review |
| `6. Operational Rules` | Coverage, pre-commit, mocking gotchas |
| `7. Session Learnings` | Codebase-specific discoveries |
| `8. Checklist` | Example — Python-specific type annotations, pylint, coverage, mypy markers (adapt for your stack) |

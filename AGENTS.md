# AGENTS.md

Rules and principles for agents working on **this** project.

---

## 1. Core Rules

### 1.0 Document Conventions

When updating this document, append new information or sections. Do NOT delete or overwrite existing content unless explicitly directed. Always ask before making structural changes. When in doubt, keep it.

### 1.1 Forbidden Patterns

The following are **strictly prohibited**:

- Hardcoded secrets, API keys, or credentials.
- `print()` statements in production code (use `logging` instead).
- `except Exception:` bare excepts.
- `eval()`, `exec()`, `__import__()` at any level.
- `*` imports (`from x import *`).
- Mutating a list while iterating over it.
- Blocking I/O (`requests`, synchronous HTTP clients) inside async functions.
- Bypassing the auth middleware.

### 1.2 Forbidden — Security

Follow the [OWASP Top 10](https://owasp.org/www-project-top-10/) for every piece of code written:

- Every route MUST pass through authentication middleware.
- Never store plaintext secrets. Use `os.environ` / `pydantic-settings`.
- Use parameterized queries. Validate and sanitize all user input via Pydantic models.
- File uploads must pass whitelist + MIME validation.
- All pydantic-settings defaults must be production-safe. No `DEBUG=true` in non-local configs.
- Implement secure token verification. Reject tokens with weak algorithms.
- Log at structured JSON level. Strip PII before logging.
- Validate all outbound tool URLs against an allowlist. Disallow `file://`, `gopher://`, `dict://` schemes.

### 1.3 Forbidden — Git Operations

- **Never rebase under any circumstance without explicit agreement from the user.** Never assume your decision is correct.
- Never force push.

### 1.4 Core Principles

- **DRY**: Extract repeated logic into functions, classes, or utilities. Centralize configuration in `config.py`. Reuse SSE envelope formatter, error handler, and auth middleware across endpoints. No copy-paste code blocks greater than three lines.
- **KISS**: Prefer simple, readable code over clever solutions. If a solution requires more than three levels of indentation or a helper function with more than 10 lines, reconsider it.
- **YAGNI**: Do NOT build features, abstractions, or configurations not required by the current spec. No generic "future-proof" wrappers. Ad-hoc solutions are acceptable as long as they serve a present requirement.
- **Single Responsibility**: Each module, class, and function must have one reason to change.
- **Open/Closed**: Extend via composition — not by modifying existing logic.
- **Dependency Inversion**: Depend on abstractions (protocols / ABCs) for external services. Inject implementations.

---

## 2. Project Context

Description here.

### 2.0 Expected Project Layout

```
directory structure here
```

Misc details here.

### 2.1 Quick Commands

| Command    | Purpose |
|------------|---------|
| `make ...` | ...     |

---

## 3. Python Conventions

### 3.1 Language & Tooling

- **Python**: 3.13+ (`from __future__ import annotations`)
- **Package manager**: `uv` — the **only** supported package manager. Never use `pip`.
- **Type checking**: `mypy` (strict mode)
- **Formatting**: `black` (line-length 88), **isort** (profile: `black`)
- **Linting**: `pylint` (defaults + mads-specific disables in `pyproject.toml`)
- **Testing**: `pytest` + `pytest-asyncio`
- **Git hooks**: `pre-commit` (manages isort, black, pylint, mypy)

### 3.2 Style

- Use 2 spaces for indentation. No tabs. Maximum line length: 100 characters.
- Follow [PEP 8](https://peps.python.org/pep-0008/) and [PEP 484](https://peps.python.org/pep-0484/).
- All public functions and classes MUST have type hints and a docstring (Google or Sphinx style).
- Private functions prefixed with `_`.
- Functions: `snake_case`. Constants: `UPPER_SNAKE_CASE`.

### 3.3 Error Handling

- Raise typed exceptions. Define domain-specific exceptions in `mads/errors.py`.
- Catch at the boundary (FastAPI exception handler), not inside logic.
- Never swallow exceptions silently.

### 3.4 Async

- Use `async def` and `await` consistently. Never mix blocking I/O in async contexts.
- All FastAPI route handlers that perform I/O MUST be `async def`.

### 3.5 Testing

- Each public function or class must have at least one test.
- Tests live in `tests/unit/` for unit tests and `tests/integration/` for integration tests.
- Mock external services — no real API calls in tests.
- Test filenames inherit the parent directory structure with `_` as delimiters to avoid filename collisions.
  - `app/graphs/assistant_graph.py` → `test_graphs_assistant_graph.py`
  - `app/tools/workspace.py` → `test_tools_workspace.py`

---

## 4. Framework Conventions

### 4.1 FastAPI / API

- Assistant endpoint: `POST /` (SSE streaming). Upload: `POST /upload`.
- Validate all input with Pydantic models. Include `request_id` in response headers.
- Middleware order (critical): `CORS → Security Headers → Request ID → Rate Limit → Auth → Route Handler`

### 4.2 LangGraph

- State must be a `TypedDict`; never use plain `dict`.
- Use `Annotated` with `AddMessages` for message history. Keep state minimal.
- Each node is a plain `async def` function receiving the state.
- Return a dict of state updates. No side effects outside the returned dict unless logged.
- Register tools with `@tool` decorator or `create_tool`. Every tool must declare input schema with clear `str` fields.
- Tools must NOT perform I/O without timeouts.

### 4.3 Auth Modes

- Modes: `jwt`, `apikey`, `none` (dev).
- API key auth uses `auth_api_key` from `SecuritySettings`. Env var: `AUTH_API_KEY`.
- JWT auth uses JWKS endpoint for verification. Enforce audience and issuer claims.

---

## 5. Git Conventions

### 5.1 Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add file upload endpoint with whitelist validation
fix: correct JWT audience claim validation
docs: update AGENTS.md with new config variables
test: add graph node unit tests for file_processor
chore: pin all dependencies in pyproject.toml
```

### 5.2 Branching

- Main branch is `main`.
- Feature branches: `feat/<short-desc>`. Bug fixes: `fix/<short-desc>`.

### 5.3 Code Review

- All changes require at least one other reviewer (automated checks are mandatory but not sufficient).
- No merging without passing CI (lint → type-check → test).
- PR descriptions must reference related items from design documents.

---

## 6. Operational Rules

Session learnings — critical gotchas that affect how code must be written and tested.

### 6.1 Coverage

The pre-commit hook enforces **100% code coverage** via `coverage report --fail-under=100`. Every new function or class needs test coverage. No exceptions.

```bash
.venv/bin/coverage run -m pytest tests/ -q --tb=no
.venv/bin/coverage report -m            # file-level missing lines
```

### 6.2 Pre-commit Hook and coverage.txt

The `cover` pre-commit hook runs tests then regenerates `coverage.txt`. If the hook modifies a staged file, `git commit` fails. Always `git add -A` and `git commit --amend -C HEAD` after a failed commit from a modified `coverage.txt`.

### 6.3 Pre-commit Runs Tests

The pre-commit hook runs `uv run pytest tests/ && uv run coverage report` in addition to linting/type-checking. A commit can fail due to test failures or insufficient coverage, not just lint or mypy.

### 6.4 Mocking Settings

The `settings` singleton from `config.py` is the single source of configuration. When patching:

- Patch where it's **imported**, not where it's defined.
- Use `patch("config.settings")` for settings read in `assistant_graph.py`.
- Use `patch("tools.<module>.settings")` for settings read in `tools/` modules.
- Mock entire settings objects rather than using `MagicMock` which can be truthy.

```python
# Wrong: MagicMock is truthy
with patch("tools.workspace.settings") as mock:
    mock.settings.tools.workspace_read_only = False  # Missing .tools!

# Right: Explicitly set all needed attributes
with patch("tools.workspace.settings") as mock_settings:
    mock_settings.tools.workspace_max_read_size = 50
    mock_settings.tools.workspace_read_only = False
```

### 6.5 Mocking MongoDB

When mocking MongoDB, the mock chain must preserve async behavior. All of `.find().limit().sort()` must return mocks with async `.to_list()` capability.

```python
def _make_db_mock(collection_names, doc_counts, docs):
    db = MagicMock()
    db.list_collection_names = AsyncMock(return_value=collection_names)
    for name in collection_names:
        _cursor = AsyncMock(to_list=AsyncMock(return_value=docs.get(name, [])))
        coll = MagicMock()
        coll.find = MagicMock(return_value=MagicMock(
            limit=MagicMock(return_value=MagicMock(
                sort=MagicMock(return_value=MagicMock(to_list=_cursor.to_list)),
                to_list=_cursor.to_list,
            )),
        ))
        db.__getitem__ = lambda self, n: coll
    return db
```

### 6.6 Unreachable Code

Code that can never execute is a smell. Remove dead code to avoid coverage gaps and confusion.

---

## 7. Session Learnings

Discovery notes about the codebase.

### 7.1 README is the source of truth for project layout

The `README.md` may show a more up-to-date project structure (e.g., additional middleware modules, tool files). When in doubt, use it to verify the layout in section 2.0.

---

## 8. Checklist Before Marking a TODO Complete

- [ ] All type annotations present (no `Any` unless justified and documented).
- [ ] Docstrings on all public APIs.
- [ ] Unit tests written and passing.
- [ ] Integration tests for API endpoints.
- [ ] `pylint`, `isort`, `black`, and `mypy` pass via pre-commit hooks.
- [ ] `mypy` or `pyright` reports zero errors on changed files.
- [ ] No hardcoded secrets or credentials introduced.
- [ ] Environment variable configuration used (no config file logic).
- [ ] 100% code coverage maintained (pre-commit will enforce this).
- [ ] Threat model considerations addressed in PR description.

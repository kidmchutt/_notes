# [Project Name] — GitHub Copilot Instructions

This file contains instructions for GitHub Copilot to follow when generating code suggestions in this repository. It should be updated as the first part of the development process, and then updated as the project evolves.

These instructions apply to every file in this repository.
Read them before suggesting any code, refactor, or new feature.

---

## Project Overview

This section is to be updated as the project evolves.

## Repository Layout

This section is to be updated as the project evolves.

## Architecture Rules

### Layered architecture — STRICT separation of concerns

This section is to be updated as the project evolves.

Example of the intended architecture:
```
1. **`db/models.py`** — SQLAlchemy ORM model definitions only. No logic.
2. **`db/engine.py`** — Engine creation, session factory, DB path resolution. No business logic.
3. **`services/`** — ALL business logic and database access. The only layer that queries the DB.
4. **`gui/`** — Presentation only. Widgets call service methods; they NEVER import SQLAlchemy or query the DB directly.
5. **`cli.py`** — Thin adapter that calls service methods and formats output. No business logic.
```

> If a widget needs data, it calls a service function.
> If a service needs the DB, it uses the session from `db/engine.py`.
> Nothing else talks to the DB.

---

## Code Conventions

### 1. Docstrings — REQUIRED on all public methods and classes

Every public class and every public method (including `__init__` if non-trivial) **must** have a docstring. Use the Google docstring style.

Every module **must** have a top-level docstring describing its purpose and any important implementation notes.

```python
def get_all_tasks(
    self,
    search: str = "",
    priority: int | None = None,
    due_before: date | None = None,
    due_after: date | None = None,
) -> list[Task]:
    """Retrieve all tasks matching the given filters.

    Args:
        search: Case-insensitive substring match against task title.
        priority: Filter to tasks with exactly this priority level (1/2/3).
        due_before: Include only tasks with due_date <= this date.
        due_after: Include only tasks with due_date >= this date.

    Returns:
        List of Task ORM objects ordered by due_date ascending, then priority descending.
    """
```

Private helpers (prefix `_`) may use shorter one-line docstrings but must still have one.

### 2. Extensibility — design for future additions

- Prefer composition over hardcoded behavior.
- Use constants or enums for magic values (priority levels, status strings, etc.).
- Keep widget classes small and focused; use signals/slots for inter-widget communication rather than direct method calls.
- Do not hardcode strings visible to the user — use a constants module so future i18n is possible.
- Service methods should accept optional keyword arguments rather than growing separate methods for every variant.

### 3. Type annotations — REQUIRED

All function signatures must include type annotations, including return types.

```python
def toggle_done(self, task_id: int) -> Task:
    ...
```

### 4. Code style

- Formatter: **black** (default settings, line length 88)
- Linter: **ruff** (default ruleset)
- Do NOT introduce style changes unrelated to the task at hand.

### 5. SQLAlchemy sessions

- Sessions are always opened via the context manager from `db/engine.py`.
- Never leave a session open outside of the scope that needs it.
- Use `expire_on_commit=False` so detached objects remain usable in the GUI layer.

```python
with get_session() as session:
    task = session.get(Task, task_id)
    ...
```

### 6. Dates

- Store dates in SQLite as ISO 8601 strings (`YYYY-MM-DD`).
- In Python, always use `datetime.date` objects, never raw strings.
- Conversion happens at the ORM boundary (SQLAlchemy type decorator if needed).

### 7. Tests

- Create tests for all new service methods and any non-trivial GUI logic before development of the method/widget itself begins.
- All tests use an **in-memory SQLite DB** (`sqlite:///:memory:`).
- Fixtures are defined in `conftest.py`; do not duplicate setup logic.
- Every new service method must have a corresponding test before the phase is marked done.
- Use `pytest`; do not use `unittest` directly.

### 8. PySide6 signals and slots

- Define custom signals as class attributes using `Signal(...)`.
- Connect slots in the parent that owns both widgets; widgets themselves only emit.
- Use `Qt.ConnectionType.QueuedConnection` for cross-thread signals.

### 9. No business logic in `main.py`

`main.py` only: parses args to decide GUI vs CLI mode, bootstraps the app, and exits.

### 10. Language support for the application should be designed with future internationalization (i18n) in mind, even if only English is supported throughout development. This means:
- All user-facing strings should be defined in a central constants module (e.g. `gui/strings.py`) rather than hardcoded in widgets.
- The constants should be organized by widget or feature for easy future translation.

### 11. Screen reader support — REQUIRED on all user-facing widgets. Use `setAccessibleName()` and `setAccessibleDescription()` as appropriate.

### 12. GUI design principles

- Follow the layered architecture strictly: GUI widgets call service methods; they do NOT import SQLAlchemy or access the DB directly.
- Keep widgets focused and reusable: a widget should ideally represent a single concept (e.g. a TaskItem) and not contain unrelated logic.
- Widgets should emit signals for user actions, and the parent widget or main window should connect those signals to service calls. This keeps the GUI decoupled from the business logic.
- Widgets should be ready for future extensions: for example, a TaskItem widget should be designed so that adding a new field (e.g. "tags") in the future would not require a complete rewrite of the widget.
- For i18n readiness, see Rule #10 — all user-facing strings must come from a central constants module.

### 13. Logging

- Use `logging.getLogger(__name__)` at the top of each module; never use `print()` for diagnostic output in production paths.
- Configure the root logger (level, format, handlers) only in `main.py` or a dedicated `logging_config.py`; other modules must not call `logging.basicConfig()`.
- Use appropriate log levels: `DEBUG` for internal state, `INFO` for significant operations, `WARNING` for recoverable issues, `ERROR` or `CRITICAL` for failures.

### 14. Git and commit conventions

- Use [Conventional Commits](https://www.conventionalcommits.org/) format: `type(scope): description` (e.g. `feat(gui): add task filter bar`).
- Common types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`.
- Keep commits focused: one logical change per commit.
- Branch naming: `feature/<short-description>`, `fix/<short-description>`.

### 15. Diagrams — prefer Mermaid over ASCII/text art

- Whenever a diagram is needed in documentation (architecture overviews, flow charts, entity-relationship diagrams, state machines, sequence diagrams), use a fenced `mermaid` code block rather than ASCII/text-based representations.
- Mermaid diagrams go in `.md` files under `docs/` (or inline in `README.md` / `PLAN.md` where appropriate).
- Keep diagram source in the same file as the prose that references it — do not store diagrams in separate image files unless a rendered export is explicitly requested.
- Supported diagram types to prefer: `flowchart`, `sequenceDiagram`, `erDiagram`, `stateDiagram-v2`, `classDiagram`.

---

## What NOT to Do

- Do NOT import `sqlalchemy` in any `gui/` file.
- Do NOT use `print()` for diagnostic output in production paths — use `logging` (see Rule #13).
- Do NOT add dependencies without updating both `requirements.txt` and this file's dependency list.
- Do NOT break the portable single-file build contract: no hard-coded absolute paths; use `platformdirs` or relative-to-executable resolution.
- Do NOT add GUI frameworks other than PySide6.
- Do NOT use `PyQt6` — it has a different license; use `PySide6` only.
- Do NOT modify or replace any SVG icon file without first showing the user the **current** SVG source and the **proposed** new SVG source, and waiting for explicit confirmation before making the change. If unicode emojis can be used instead of custom icons for a given button/action, prefer that for simplicity and consistency with system UI.
- When adding buttons or actions that match an existing UI pattern (e.g. confirm/save, cancel/close, warning/alert), always reuse the same icon already used for that pattern elsewhere in the app. Check `gui/icons/` and existing dialogs before choosing an icon — consistency takes priority over novelty.

---

## Dependency List

FOSS libraries and tools are preferred. Any new dependency must be added to the appropriate table below, along with a brief description of its purpose. This helps maintain a clear overview of the project's dependencies and their roles.

### Runtime (`requirements.txt`)

This section is to be updated as the project evolves. Every new runtime dependency must be added to the table below, along with a brief description of its purpose.

| Package | Version | Purpose |
|---|---|---|
| — | — | — |

### Dev / Build (`requirements-dev.txt`)

This section is to be updated as the project evolves. Every new dev/build dependency must be added to the table below, along with a brief description of its purpose.

| Package | Version | Purpose |
|---|---|---|
| — | — | — |

---

## Phase Status (update when phases complete)

This section is to be updated as the project evolves.

| Phase | Description | Status |
|---|---|---|
| — | — | — |

---

## Cross-Platform Build Notes

Every applicable note about building and packaging the app for different platforms (Windows, macOS, Linux) should go here. This includes any platform-specific dependencies, build tools, or packaging formats to be aware of.

This section is to be updated as the project evolves.

---

## Basic repository setup

New repository should start with a basic setup including:
- `main.py` — entry point that bootstraps the app and decides between GUI and CLI mode based on command-line args.
- `README.md` — with a Getting Started section that describes how to run the app in both GUI and CLI modes.
- `requirements.txt` and `requirements-dev.txt` — with the initial dependencies (...) listed. This should be updated as new dependencies are added during development.
- `docs/PLAN.md` — with the initial project plan and milestones outlined. This should  be updated as the project evolves and milestones are completed.
- .venv/ — with the virtual environment for development. This should be updated as new dependencies are added.
- docs/ — with any initial documentation files (e.g. architecture overview, design decisions, etc.). This should be updated as the project evolves and new documentation is created.
- if necessary, create _ai/ directory with any AI-related utilities or scripts, and update this instructions file to include guidelines for their use. Subagent scripts should also follow the same documentation and code conventions outlined in this file.
- naming conventions — use lowercase with underscores for files and directories (e.g. `db/`, `services/`, `gui/`), and CamelCase for classes (e.g. `TaskItemWidget`).
- naming conventions for test files — use `test_` prefix (e.g. `test_services.py`) and place them in a `tests/` directory.
- initial SQLAlchemy setup with an in-memory SQLite database for testing, and a file-based SQLite database for production. This should be implemented in `db/engine.py` and the ORM models defined in `db/models.py`.
- naming conventions for database models — use singular nouns (e.g. `Task` rather than `Tasks`) and define them in `db/models.py` with SQLAlchemy ORM.
- naming convention for subagent scripts should contain subagent name and purpose (e.g. `subagent_data_importer.py`), and they should be placed in the `_ai/` directory.

## Documentation Updates

After implementing any new feature, behaviour change, or bug fix, **always** update the relevant documentation in the same response before returning control to the user:

- **`RELEASE_NOTES.md`** — add a bullet under the current `## vX.Y.Z — In development` section describing the change. Keep entries brief (one line each). This is the primary changelog for users.
- **`README.md`** — add or update the checklist item in the relevant phase/version section; update any prose sections (Getting Started, CLI Usage, Known Issues, etc.) that describe the changed behaviour.
- **`docs/PLAN.md`** — mark affected task items as complete (`- [x]`) and update the Milestones Summary table if a phase status changes.
- **`MEMORY.md`** — update the last-updated date and current-phase note; revise the Pending Work Items section.
- **`ai/copilot-instructions.md`** — update the Phase Status table if a phase moves to ✅ Done.

Do **not** create a separate changelog file. Documentation lives only in the files above. Every new documentation created on purpose should live in the /docs/ directory, but documentation related to the user-facing app should be in the files above.

---

## Response Convention

After completing a task and returning control to the user, always end the reply with:

`✓ Done — ready for next task.`

This applies even in the planning phase when no code has been generated yet. It signals that the current task is fully complete and the user can proceed to the next one.

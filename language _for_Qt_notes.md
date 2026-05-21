For a PySide6 app, the native Qt pipeline is the right fit. Here's the recommended approach:

---

## Qt `QTranslator` + `lupdate`/`lrelease` pipeline

### How it works

1. **Mark strings** in all GUI code with `self.tr("text")` (inside `QObject` subclasses) or `QCoreApplication.translate("Context", "text")` (anywhere):
   ```python
   self._new_btn.setText(self.tr("+ New Task"))
   ```

2. **Extract** all marked strings into an XML `.ts` file:
   ```bash
   pyside6-lupdate src/ -ts translations/tasksmanager_pl.ts
   ```

3. **Translate** using Qt Linguist (GUI tool) or any `.ts`-aware editor / Weblate

4. **Compile** to a binary `.qm` file:
   ```bash
   pyside6-lrelease translations/tasksmanager_pl.ts -qm translations/tasksmanager_pl.qm
   ```

5. **Load** at startup in app.py, after `QApplication` is created:
   ```python
   translator = QTranslator()
   locale = QLocale.system().name()           # e.g. "pl_PL"
   if translator.load(f"tasksmanager_{locale}", ":/translations"):
       app.installTranslator(translator)
   ```

---

## What needs to change in this codebase

| File | Change |
|---|---|
| `constants.py` | Remove English string constants. Move them to `tr()` call-sites, or keep as translation keys resolved via `QCoreApplication.translate()`. |
| All `gui/` widgets | Replace string literals with `self.tr("...")` |
| app.py | Add `QTranslator` loading before `MainWindow` is created |
| pyproject.toml / build | Add `translations/*.qm` to PyInstaller data files |

### The `constants.py` complication

The current module holds strings like `LABEL_NEW_TASK = "+ New Task"` — but `lupdate` only scans for `tr()` / `translate()` call patterns. Two options:

- **Option A (simpler):** Delete the string constants; use `self.tr("+ New Task")` directly at each call-site. `lupdate` extracts them automatically.
- **Option B (keep constants):** Wrap them so `lupdate` can still find them:
  ```python
  def _tr(s: str) -> str:
      return QCoreApplication.translate("constants", s)

  LABEL_NEW_TASK = lambda: _tr("+ New Task")  # called at use-site
  ```
  Option A is cleaner and is idiomatic Qt.

---

## CLI (`click`) strings

`lupdate` won't pick up Click's CLI strings. For those, keep two options:
- Leave CLI output in English (common for developer-facing tools)
- Use Python's `gettext` in parallel just for `cli.py` (it's a thin layer, so it's manageable)

---

## Packaging

`.qm` files must be added as PyInstaller data:
```python
# tasksmanager.spec
datas=[("src/tasksmanager/translations/*.qm", "tasksmanager/translations")]
```
And optionally embed them as Qt resources via a `.qrc` file for a truly single-file binary.

---

**Bottom line:** Qt's native pipeline is mature, has great tooling (`pyside6-lupdate`, Qt Linguist), and is well-supported by PyInstaller. The main upfront cost is replacing all string literals in `gui/` with `self.tr()` — mechanical but tedious. Everything else (extraction, translation, compilation, loading) is handled by Qt tooling.

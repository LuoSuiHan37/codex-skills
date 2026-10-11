# Packaging lessons and references

## Local evidence

The project uses:

- `build_windows.ps1`: installs requirements and Chromium, then runs `python -m PyInstaller --noconfirm --clean UpworkJobMonitor.spec`.
- `UpworkJobMonitor.spec`: collects Playwright and PySide6 data/binaries, adds hidden imports, filters duplicate Qt/CRT DLLs, includes `runtime_hook.py`, and uses `COLLECT`, which produces an onedir bundle.
- `runtime_hook.py` and `app.py`: add the frozen bundle, `PySide6`, and `shiboken6` directories to the Windows DLL search path and log frozen startup failures.
- `src/utils/paths.py`: distinguishes `BUNDLE_ROOT` (`sys._MEIPASS`) from writable `APP_ROOT` (`sys.executable` parent).
- `src/browser/upwork_browser.py`: stores the browser profile beside the EXE and prefers installed Chrome before searching Playwright's browser cache.
- `README.md`: documents that the current artifact is a portable directory and says the target machine needs Python, Playwright, and Chromium. That statement is not compatible with the desired single-EXE/no-prerequisite outcome and must be treated as a known gap.

## Failure-to-fix map

| Symptom | Likely cause | Fix captured in the project/skill |
|---|---|---|
| Qt DLL/import error on another PC | DLLs were not collected, were in an unexpected directory, or duplicate Qt/CRT copies confused the Windows loader | collect PySide6/shiboken6, keep one consistent DLL set, register DLL directories in a runtime hook, disable UPX while diagnosing |
| Config not found after freezing | source-relative path was used for a bundled app | read resources from `_MEIPASS`/bundle root and write mutable files beside `sys.executable` |
| Playwright says browser is missing | Playwright Python package and browser binaries are separate | bundle the exact browser or explicitly install it on the target; do not treat `pip install playwright` as sufficient |
| GUI closes without a message | windowed build hides traceback | use a console build during diagnosis or write `startup_error.log` next to the EXE |
| Copying only the EXE fails | output is onedir or has external browser/runtime requirements | copy the complete onedir directory, or finish and verify a real onefile build |

## Authoritative references

- [PyInstaller runtime information](https://pyinstaller.org/en/stable/runtime-information.html): explains `sys.frozen`, `sys._MEIPASS`, and the difference between onedir and onefile runtime paths.
- [Qt for Python: PyInstaller deployment](https://doc.qt.io/qtforpython-6/deployment/deployment-pyinstaller.html): documents PySide6 deployment, Qt 6 caveats, and onedir/onefile behavior.
- [Qt for Python deployment overview](https://doc.qt.io/qtforpython-6/deployment/index.html): compares deployment tools and resource/packaging considerations.
- [Playwright Python browser installation](https://playwright.dev/python/docs/browsers): documents browser-version coupling, cache locations, and `PLAYWRIGHT_BROWSERS_PATH`.
- [Playwright Python library guide](https://playwright.dev/python/docs/library): documents using `PLAYWRIGHT_BROWSERS_PATH=0` to bundle browser binaries with a PyInstaller executable and warns about the resulting size.
- [Playwright BrowserType API](https://playwright.dev/python/docs/api/class-browsertype): documents `executable_path` and the risk that custom executables may differ from Playwright's bundled browser.

These references support the packaging decisions; they do not prove that this particular application works on every Windows version. That still requires artifact-level testing.

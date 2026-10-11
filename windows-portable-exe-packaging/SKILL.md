---
name: windows-portable-exe-packaging
description: Package Python desktop applications for Windows with PyInstaller or PySide6 deployment, diagnose failures on other PCs, and produce a verified portable or single-file executable when the dependency model allows it.
metadata:
  short-description: Build and verify portable Windows EXEs
---

# Windows portable EXE packaging

Use this skill when a Python desktop application must be distributed to another Windows computer, especially when it uses PySide6/Qt, Playwright, SQLite, pandas/openpyxl, native DLLs, or external browser runtimes.

## Outcome and honesty requirement

The deliverable must state which distribution was actually verified:

- **onedir portable**: copy the complete output directory, including `_internal` and all collected data.
- **onefile**: copy one `.exe`; bundled files are extracted to a temporary `_MEIPASS` directory at startup.
- **single-file user experience with external runtime**: one `.exe` still depends on something installed separately and does not satisfy the user's fully portable requirement.

Do not call an onedir build “single-file”. Do not claim “works on any Windows” without testing on a clean target machine or VM. Record architecture, Windows version, Python/package versions, and whether Microsoft VC++ runtime, a browser, network access, or write permissions are required.

## Preferred workflow

1. Inspect the entry point, requirements, spec/build scripts, resource directories, runtime path code, and existing failure logs before changing packaging.
2. Build from the intended Windows architecture using `python -m pip` and pinned or tested dependency versions. Use a clean virtual environment when possible.
3. Prefer a spec file for non-trivial applications. Collect package data and binaries deliberately; add hidden imports only for imports that static analysis cannot see.
4. For PySide6, collect the Qt/PySide6 and shiboken6 binaries, plugins, translations, and required data. If Windows loader resolution is unreliable, add a runtime hook that registers the bundle, PySide6, and shiboken6 directories with `os.add_dll_directory()` and `PATH`.
5. Separate read-only bundled resources from writable user data. Resolve bundled resources through `_MEIPASS`; resolve databases, profiles, logs, exports, and user configuration beside the executable through `sys.executable` when frozen. Never write into `_MEIPASS`.
6. For Playwright, decide explicitly whether to use a bundled browser or an installed Chrome/Edge. A Python package alone is not the browser. For a truly self-contained build, install only the required browser with `PLAYWRIGHT_BROWSERS_PATH=0`, collect it, resolve it from the frozen bundle, and test startup, profile creation, navigation, and browser shutdown. This greatly increases output size and can still be constrained by browser/website policy.
7. Keep the first failing startup diagnosable: enable a console build during troubleshooting or write a frozen-mode `startup_error.log` next to the executable. Do not leave silent `console=False` failures uninstrumented.
8. Test the exact artifact, not the source tree: launch from a path containing spaces and a non-ASCII character, with no project Python environment active, and with a fresh writable data directory. Test both normal launch and the first use of every native/runtime-dependent feature.
9. Inspect the output for missing DLLs, data files, hidden imports, Qt plugins, browser executables, and accidental absolute paths. Rebuild with `--clean` after spec changes.

## Known failure patterns and fixes

- **“The EXE opens on the build PC but not elsewhere”**: the build was actually an onedir package, a dependency was not collected, or the target lacks an external runtime. Copy the whole output directory for onedir; otherwise collect the missing dependency and retest on a clean machine.
- **Qt/PySide6 starts with a DLL error**: duplicate or misplaced Qt/CRT DLLs can cause Windows to load the wrong copy. Keep a consistent Qt binary set, collect shiboken6, register bundle DLL directories early with a runtime hook, and avoid UPX compression while diagnosing.
- **Config/data works from source but fails when frozen**: source-relative paths do not describe a frozen application. Use a bundled-resource path with `_MEIPASS` fallback and a writable path based on `sys.executable`.
- **Playwright reports that Chromium is missing**: `pip install playwright` does not install browser binaries. Either install the exact browser separately and document that prerequisite, or bundle the required browser and use a frozen-mode executable path.
- **A browser exists but the target feature fails**: browser revision mismatch, wrong `PLAYWRIGHT_BROWSERS_PATH`, permissions, profile locking, or website/Cloudflare behavior may be involved. Log the selected browser path and profile path; test with a new profile and preserve a clear user-facing error.
- **No useful error appears**: `console=False` hides the traceback. Temporarily build windowed=False or write exceptions to a log beside the EXE before investigating packaging changes.

## Project-specific lessons captured here

The source project that produced this skill uses `UpworkJobMonitor.spec`, `runtime_hook.py`, `src/utils/paths.py`, and `build_windows.ps1`. Its successful compatibility measures include `collect_all("playwright")`, `collect_all("PySide6")`, explicit pandas/openpyxl/winotify hidden imports, Qt/shiboken DLL relocation, `os.add_dll_directory()`, and separate `APP_ROOT`/`BUNDLE_ROOT` paths. Its verified output is `dist/UpworkJobMonitor/` with `_internal/`, not only `UpworkJobMonitor.exe`.

Its current Playwright code prefers an installed Google Chrome and falls back to a Playwright Chromium cache. Therefore the current build still requires an external browser on a new computer and does **not** yet meet the one-EXE/no-prerequisite goal. To close that gap, change the browser resolution and build contents, then validate on a clean Windows machine; do not simply copy the current EXE.

## References

Read [references/packaging-lessons.md](references/packaging-lessons.md) when documenting the build, troubleshooting a target-machine failure, or deciding whether onefile is realistic. It contains the local evidence and authoritative references used for this skill.

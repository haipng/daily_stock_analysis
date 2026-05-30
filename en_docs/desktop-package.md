# Desktop App Packaging (Electron + React UI)

This project can be packaged as a desktop application using Electron as desktop shell and React UI from `apps/dsa-web` as interface.

## Architecture Overview

- React UI (Vite build) hosted by local FastAPI service
- Electron auto-starts backend service on launch, loads UI after `/api/health` ready
- Windows portable/install modes save user configuration file `.env` and database in exe sibling directory; macOS bundle uses Electron user data directory for runtime configuration

## Local Development

One-click startup (dev mode):

```bash
powershell -ExecutionPolicy Bypass -File scripts\run-desktop.ps1
```

Or manual execution:

1) Build React UI (output to `static/`)

```bash
cd apps/dsa-web
npm install
npm run build
```

2) Start Electron app (auto-starts backend)

```bash
cd apps/dsa-desktop
npm install
npm run dev
```

First run auto-generates `.env` from `.env.example`.

## Packaging (Windows)

### Prerequisites

- Node.js 18+
- Python 3.10+
- Enable Windows Developer Mode (electron-builder needs symlink creation)
  - Settings → Privacy and Security → Developer Options → Developer Mode

### One-Click Packaging

```bash
powershell -ExecutionPolicy Bypass -File scripts\build-all.ps1
```

Script sequentially executes:
1. Build React UI
2. Install Python dependencies
3. PyInstaller package backend
4. electron-builder package desktop app

Current Windows installer uses NSIS wizard-based flow; supports current-user install only with admin elevation disabled; during install users can manually choose target directory (e.g., non-C: drive). Installer blocks system-protected directories like `Program Files`, `Windows` at NSIS layer—"Next" button auto-disables when selecting these. After install, desktop app generates/reads `.env`, `data/stock_analysis.db` (with `data/stock_analysis.db-wal` / `data/stock_analysis.db-shm`) and `logs/desktop.log` in install directory. Recommend using default per-user install directory. If prefer not to install, still can distribute `win-unpacked` portable package.

## GitHub CI Auto-Package and Release

Repository supports auto-building desktop app and uploading to GitHub Releases via GitHub Actions:

- Workflow: `.github/workflows/desktop-release.yml`
- Trigger:
  - After pushing semantic version tag (e.g., `v3.2.12`) auto-triggers
  - Or manually trigger from Actions page with specified `release_tag`
- Artifacts:
  - Windows installer: Release attachment and local `apps/dsa-desktop/dist/` both named `daily-stock-analysis-windows-installer-<tag>.exe`
  - Windows auto-update metadata: Release also keeps `latest.yml` and `*.blockmap` for installed desktop app background download and verification; normal users don't download these. After download, when user confirms "restart install", desktop auto-stops backend, backs up runtime files, then silently runs installer.
  - Windows portable: `daily-stock-analysis-windows-noinstall-<tag>.zip`
  - macOS Intel: `daily-stock-analysis-macos-x64-<tag>.dmg`
  - macOS Apple Silicon: `daily-stock-analysis-macos-arm64-<tag>.dmg`

Recommended release flow:

1. Merge code to `main`
2. Auto-tag workflow generates version (or manually create tag)
3. `desktop-release` workflow auto-builds and adds both platform installers to corresponding GitHub Release

## Pre-Release Desktop Update Path Verification (Windows)

Desktop auto-update path depends on Windows NSIS installer artifacts, `latest.yml`, `*.blockmap` metadata. Current desktop CI doesn't cover `desktop-release` packaging artifact publishability path; pre-submit recommend following local verification:

Note: This checklist focuses on Windows NSIS installer and `electron-updater` release metadata. Current Linux cannot directly produce Windows installer and updater metadata (`latest.yml` / `*.blockmap`); such paths need Windows release executor or Windows native environment review. 

If can't complete above verification in non-Windows environment, clearly note in PR description Windows release path reviewer, review time window, and `desktop-release` artifact inspection results (release/tag and `daily-stock-analysis-windows-installer-<tag>.exe`, `latest.yml`, `*.blockmap` version consistency and downloadability).

1. First build Web static artifacts (desktop main window and settings page depend on it)

```bash
cd apps/dsa-web
npm ci
npm run lint
npm run build
```

2. Complete desktop deps, run preload tests, then execute Electron build

```bash
cd ../dsa-desktop
npm ci
npm test
npm run build
```

In Windows release review environment, can additionally execute:

```powershell
./scripts/verify-desktop-updater-artifacts.ps1 -ReleaseTag v$(node -p "require('./apps/dsa-desktop/package.json').version")
```

> Expected: If current execution environment doesn't support Windows NSIS installer generation, clearly note platform limitation in delivery description and require specified Windows release path reviewer complete that verification item.

3. Check updater metadata generated

```bash
ls -1 dist | sort
ls -1 dist/*.yml dist/*.blockmap 2>/dev/null || true
```

4. Force-align version and release attachment (can verify in Windows environment or executor producing NSIS artifacts)

```bash
RELEASE_TAG="v$(node -p \"require('./package.json').version\")"
REPO="ZhuLinsen/daily_stock_analysis"

for f in dist/*latest.yml dist/*.blockmap dist/daily-stock-analysis-windows-installer-*.exe; do
  [ -f \"$f\" ] && echo \"[FOUND] $f\"
done

if [ -f dist/latest.yml ]; then
  echo \"---- latest.yml version snippet ----\"
  grep -E \"^version:|^files:|^sha512:\" dist/latest.yml
fi

echo \"---- Release checklist (manual verify) ----\"
echo \"Release Tag: $RELEASE_TAG\"
echo \"Release URL: https://github.com/$REPO/releases/tag/$RELEASE_TAG\"
echo \"Should verify attachments contain:\"
echo \"- daily-stock-analysis-windows-installer-*.exe\"
echo \"- latest.yml\"
echo \"- *.blockmap\"
echo \"And ensure latest.yml version matches tag semantic version, path/url matches installer attachment name\"
```

5a. Recommend in PR description recording "verifiable output" (Windows):

```bash
echo "release-tag=${RELEASE_TAG}"
echo "latest.yml version:"
grep -E "^version:" dist/latest.yml
echo "latest.yml files:"
sed -n '1,80p' dist/latest.yml
echo "packaging artifacts:"
ls -1 dist/*.yml dist/*.blockmap dist/*installer*.exe 2>/dev/null | sort
```

Windows release path review checklist (executed post-PR by release team/maintainer):

- release/tag and `daily-stock-analysis-windows-installer-<tag>.exe` version numbers match
- `latest.yml`, `daily-stock-analysis-windows-installer-<tag>.exe`, `*.blockmap` appear together with same tag and downloadable
- `latest.yml` `version` matches Release tag semantic (compare after stripping `v` prefix), and `path` / `files.url` matches installer attachment name
- If missing above or `release-tag` mismatch, need to flag blocking and complete `desktop-release` packaging flow

5. Windows/NSIS artifacts and release attachment consistency manually verify in Windows environment (can manually trigger release flow), and verify runtime file retention post-upgrade:

   1. Record install-before and after in install directory: `.env`, `data/stock_analysis.db`, `data/stock_analysis.db-wal`, `data/stock_analysis.db-shm`, `logs/desktop.log` SHA256
   2. Confirm after desktop next startup, above files still exist and match install-before record
   3. If inconsistent, can check in user data directory `.dsa-desktop-update-backup` for completeness after app exit, and combine latest logs for investigation

Windows recommend using PowerShell:

```bash
Get-FileHash .env,data\\stock_analysis.db,data\\stock_analysis.db-wal,data\\stock_analysis.db-shm,logs\\desktop.log -Algorithm SHA256
```

Note: App already stops backend pre-"restart install" on Windows NSIS, backs up install-directory runtime files, then silently runs update installer, to prevent wizard hogging still-running desktop process and reduce file loss during update; if restore fails, desktop shows update install error and keeps manual download fallback path. This fix only changes Windows update install path and backend process lifecycle handling, not settings save semantics, model runtime cleanup policy, or configuration migration behavior.

## Directory Structure

Windows installer-mode only supports current-user install with admin elevation disabled; users can choose install directory in wizard; installer blocks system-protected directories at NSIS layer (Next button auto-disables when selecting), then app generates/reads `.env`, `data/stock_analysis.db` (with `data/stock_analysis.db-wal` / `data/stock_analysis.db-shm`), `logs/desktop.log` in install directory. Recommend keeping default per-user install location.

`win-unpacked` portable-mode directory structure:

```
win-unpacked/
  Daily Stock Analysis.exe    <- Double-click to start
  .env                        <- User config file (auto-generated on first start)
  data/
    stock_analysis.db         <- Database main file
    stock_analysis.db-wal     <- WAL log file (update backup/restore)
    stock_analysis.db-shm     <- WAL shared metadata file (update backup/restore)
  logs/
    desktop.log               <- Run logs
  resources/
    .env.example              <- Config template
    backend/
      stock_analysis.exe      <- Backend service
```

## Configuration File Explanation

- Windows desktop `.env` in exe sibling directory
- macOS bundle `.env`, `data/`, `logs/` in Electron user data directory, avoids losing on `.app` replacement
- First start auto-generates from `.env.example`
- When upgrading from old version, if old `.app` internal files still accessible, new version auto-migrates one-time to user data directory
- After migration or reconfiguration, subsequent versions continue using user data directory without losing on `.app` replacement
- Users need to configure `.env`:
  - `GEMINI_API_KEY` or `OPENAI_API_KEY`: Required for AI analysis
  - `STOCK_LIST`: Watchlist (comma-separated)
  - Other optional per `.env.example`

### Configuration Backup / Restore `.env`

- WebUI and desktop both show "System Settings → Configuration Backup" with `Export .env` and `Import .env` buttons
- WebUI non-desktop run needs first enabling admin auth and login completion; button disabled without auth, API returns `403`
- `Export .env` exports currently **saved** `.env` backup file; unsaved drafts on page don't export
- `Import .env` reads backup keys and merges to current config, immediately triggers reload
- Import is "key-level override" not full-file replacement: backup keys override current; absent keys unchanged
- If current page has unsaved drafts, import prompts confirmation first, avoiding mixing drafts and saved config
- Web defaults `ADMIN_AUTH_ENABLED=false`; settings page shows disabled button prompting enable auth first; desktop unaffected by this config, still directly uses backup/restore.

> Recommend: Upgrading macOS users still can `Export .env` pre-upgrade as insurance; if old `.app` already fully replaced, old internal files can't recover standalone; only importable via backup.

### Settings Page Version Info

- "System Settings → Version Info" "Desktop Version" from Electron main process `app.getVersion()`, exposed to frontend via preload bridge
- Dev `npm run dev` and package `npm run build` / installers all reuse same version injection chain, no longer maintain separate hardcoded version in `preload.js`
- `README.md` still keeps install and run entry; desktop runtime details unified in this spec document to avoid inflating intro docs

### Desktop Update Reminders

- App background-checks GitHub Releases latest official version post-main loading and semantic-version-compares with current `app.getVersion()`
- Windows NSIS installed auto-downloads new versions from built-in GitHub source; after download pops one-time reminder, user confirms then silently restarts and installs
- "System Settings → Version Info" "Desktop Update" area manually checks; if downloaded shows "restart install" action
- Windows portable, dev mode, and macOS DMG still keep "notify + jump download page" compat path, won't block desktop start on network failure
- Version check failure, GitHub API timeout, missing update metadata, or download install errors logged to `logs/desktop.log`; manual check in settings shows error state

## Common Questions

### Shows "Preparing backend..." Forever After Launch

1. Check `logs/desktop.log` for errors
2. Confirm `.env` exists and correct configuration
3. Confirm ports 8000-8100 not occupied

### Backend Start Reports ModuleNotFoundError

PyInstaller missing module during packaging; need adding `--hidden-import` in `scripts/build-backend.ps1`.

### UI Loads Blank

Confirm `static/index.html` exists; if not, need re-building React UI.

### macOS Post-Upgrade Configuration Migration

Old versions wrote `.env`, database, logs inside `.app` package. New version uses Electron user data directory, auto-migrates one-time when old `.app` files still accessible. Migration is "target not exist copy only", avoids overwriting user already-saved in new version.

If old `.app` already fully replaced, old `.env` can't auto-recover. Can import pre-upgrade exported `.env` in "System Settings → Configuration Backup", or reconfigure post-migration once; subsequent versions continue using user data directory without losing on `.app` replace.

## Distribution to Users

Windows now has two distribution ways:

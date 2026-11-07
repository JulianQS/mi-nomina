## Why

Mi Nómina is used to track a household employee's pay, vacaciones and statutory obligations — things that come up away from the desk (paying a quincena, logging an absence, checking when prima is due). Today the app only runs as `index.html` opened on a computer. An iPhone version lets the employer check and record things on the spot, while keeping the project's core properties: no server, no build step, data stays on the device.

Delivering it as an installable Progressive Web App (Add to Home Screen from Safari) reuses the existing single-file app instead of creating a second codebase, and avoids Xcode, App Store review, and a developer account.

## What Changes

- Make the app installable on iOS: web app manifest, home-screen icons, and the iOS-specific `<meta>`/`<link>` tags so it launches full-screen with its own icon and name ("Mi Nómina").
- Make the app work offline after first load: a service worker caches the app shell, and React, ReactDOM and Babel Standalone are vendored locally at pinned versions (they currently load unpinned development builds from unpkg.com).
- Adapt the layout for iPhone: respect safe-area insets (notch, Dynamic Island, home indicator) on the header and bottom tab bar, prevent Safari's auto-zoom on form inputs, and size touch targets for fingers.
- Make moving data between devices reliable on iOS: export uses the iOS share sheet (Save to Files, AirDrop, Messages) when available; import accepts JSON files from the Files app, validates the file shape, and asks for confirmation before replacing existing data.
- Ask the browser to treat app storage as persistent, and surface a "last backup" reminder so iOS storage eviction can't silently lose data.
- Host the app on a static HTTPS origin (required for service workers and installation). Opening `index.html` directly from disk keeps working, without offline/install features.

## Capabilities

### New Capabilities
- `pwa-install`: Installability on iOS — manifest, icons, standalone display, app name/theme, and how the app behaves when launched from the home screen.
- `offline-support`: Service worker caching of the app shell and vendored libraries, versioned cache updates, and graceful behavior when opened via `file://`.
- `mobile-layout`: iPhone ergonomics — safe-area insets, no input auto-zoom, touch target sizing, standalone-mode status bar.
- `data-portability`: Export via share sheet / download, validated and confirmed import, persistent-storage request, and backup reminder.

### Modified Capabilities
<!-- None: existing payroll, prima, cesantías, and dashboard requirements are unchanged. -->

## Impact

- **`index.html`**: new head tags, service-worker registration, local `<script>` sources, safe-area/input CSS, changes to `exportJSON` / `importJSON` in `App`, backup-reminder UI.
- **New files**: `manifest.webmanifest`, `sw.js`, `icons/` (PNG app icons), `vendor/` (pinned React 18, ReactDOM 18, Babel Standalone production builds).
- **`.gitignore`**: currently ignores `*.json`; the manifest uses the `.webmanifest` extension so it is not swallowed, and no other new file is JSON.
- **Hosting**: requires a static HTTPS host (e.g. GitHub Pages). The repo currently has no remote — one must be created.
- **Data**: no schema change to `mi_nomina_db`. On iOS, the installed home-screen app has storage separate from Safari tabs, so the initial data must be brought in via import.
- **Docs**: README and CLAUDE.md get install, hosting and data-transfer instructions; CLAUDE.md's "update each January" steps add bumping the service-worker cache version.

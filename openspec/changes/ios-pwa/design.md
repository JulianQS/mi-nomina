## Context

Mi Nómina is a single `index.html`: React 18 + Babel Standalone compile JSX in the browser, Tailwind is a precompiled CSS subset inlined in a `<style id="tailwind-generated">` block (vendored in 1de347a after the Tailwind CDN was DNS-blocked), and all data lives in `localStorage["mi_nomina_db"]`. React, ReactDOM and Babel are still loaded from `unpkg.com` with unpinned versions (`react@18`, `@babel/standalone` latest) and React is the **development** build.

The UI is already mobile-shaped (`max-w-lg` column, fixed bottom tab bar, bottom sheets), so most of the work is platform integration rather than a redesign.

Relevant iOS facts that shape the design:
- Service workers and installation require a secure context (HTTPS or `localhost`); they never run under `file://`.
- A home-screen web app on iOS has **its own storage container**, separate from Safari tabs. Data entered in Safari does not appear in the installed app, and vice versa.
- iOS Safari can evict script-writable storage for sites not used in 7 days; home-screen apps are exempt from that rule, but storage can still be lost if the app is deleted or the device is low on space.
- iOS zooms into inputs whose font size is under 16px. The app's inputs use `text-sm` (14px).
- `<a download>` with a blob URL works poorly in standalone mode; `navigator.share({ files })` is supported since iOS 15.
- `<input type="file" accept=".json">` can grey out JSON files in the Files picker; MIME types should be listed too.

No git remote exists yet; there is no hosting today.

## Goals / Non-Goals

**Goals:**
- Installable on iPhone from Safari, launches full-screen with its own icon.
- Fully usable offline after first load.
- Correct layout on notched / Dynamic Island iPhones in standalone mode.
- A safe, obvious path to move data between the desktop and the phone, and to keep backups.
- Keep the zero-build, single-file nature of the app and keep `file://` use working.

**Non-Goals:**
- App Store / TestFlight distribution (Capacitor or native would be a later change).
- Cloud sync, accounts, or any backend.
- Push notifications or reminders for due obligations (possible follow-up; iOS web push needs a server).
- Android-specific polish (the manifest helps Android too, but it's not tested).
- Changes to payroll, prima, cesantías or dashboard behavior.

## Decisions

### 1. PWA instead of a native wrapper or rewrite
Reuses the existing code with no second codebase, no Xcode, no $99/yr developer account, no App Store review. Calculations stay in one place.
*Alternatives:* Capacitor (App Store distribution and native APIs, but adds npm + Xcode and a build step); SwiftUI rewrite (best native feel, but duplicates every Colombian payroll rule). Both are still possible later, and a Capacitor shell could load this same `index.html`.

### 2. Vendor pinned production builds into `vendor/`
`vendor/react.production.min.js` (18.3.1), `vendor/react-dom.production.min.js` (18.3.1), `vendor/babel.min.js` (a pinned 7.x Standalone). This removes the last third-party runtime dependency (same reason Tailwind was vendored), makes the service worker precache same-origin files instead of opaque cross-origin responses, and stops a future unpinned Babel or React release from breaking the app silently.
*Alternatives:* cache the unpkg URLs in the service worker. Rejected: opaque responses can't be checked, versions still float, and the DNS-blocking issue still applies on first load.

### 3. Keep runtime Babel compilation (no build step)
Compiling ~1,200 lines of JSX on an iPhone takes a few hundred ms. That's acceptable, and it keeps "edit `index.html`, reload" working.
*Alternative:* precompile JSX to plain JS. Faster startup, but adds a build step and a second artifact to keep in sync. If startup feels slow on device, revisit in a follow-up.

### 4. Hand-written cache-first service worker
`sw.js` (~50 lines, no Workbox): a `CACHE_VERSION` constant, precache list on `install`, delete old caches on `activate`, cache-first `fetch` for same-origin GETs with network fallback. It does not call `skipWaiting()` automatically. Instead the page listens for a waiting worker and shows a "Nueva versión disponible — recargar" prompt that posts `SKIP_WAITING` and reloads on `controllerchange`. This avoids swapping code under a user mid-edit.
Registration lives in a small plain `<script>` (not the Babel block), guarded by `'serviceWorker' in navigator && location.protocol.startsWith('http')`, so `file://` is untouched.
*Alternatives:* network-first (slower and breaks with flaky signal); Workbox (adds a dependency, too much for 6 files).

### 5. Manifest named `manifest.webmanifest`
`.gitignore` has `*.json` to keep personal data exports out of git. `.webmanifest` is the standard extension, avoids the rule, and needs no `.gitignore` exception that could someday let a data export slip into the repo.

### 6. New CSS goes in the plain `<style>` block, not Tailwind classes
The inlined Tailwind build only contains classes already used. New utilities (e.g. `pb-[env(...)]`) would silently do nothing. Safe-area padding, the 16px input rule (`@media (pointer: coarse) { input, select, textarea { font-size: 16px } }`), `-webkit-tap-highlight-color`, and `overscroll-behavior` go into the hand-written `<style>` block next to the existing `select` rule. Small semantic class names (`.safe-top`, `.safe-bottom`) are added to the header, tab bar and `BottomSheet`.
*Alternative:* regenerate the Tailwind subset. Possible, but the regeneration process isn't documented in the repo, and plain CSS is clearer for these few rules.

### 7. Export: Web Share with download fallback
`exportJSON` builds a `File` named `mi_nomina_YYYY-MM-DD.json`. If `navigator.canShare?.({ files: [file] })`, call `navigator.share`; ignore `AbortError` (user cancelled). Otherwise use the existing `<a download>` path. On success, write `mi_nomina_last_export` (ISO date) to `localStorage` under its **own key**, so it isn't part of exports and doesn't travel between devices.

### 8. Import: validate, migrate, confirm
Pull the legacy `pto_db` migration out of `loadDB` into `migrateDB(obj)`, used by both `loadDB` and import. Import then: parse → `migrateDB` → shape check (`employees` is a non-empty array; each has `id`, `name`, `salary_history[]`) → `confirm("Reemplazar los datos actuales con N empleada(s) del archivo?")` → `setDb`. If `active` doesn't match an employee, fall back to the first one. File input `accept="application/json,.json,text/plain"`.
`window.confirm` is used rather than a custom sheet: it works in standalone iOS and matches the existing `alert` use.

### 9. Install hint and backup reminder as small banners
Both are conditional banners in `App`, above the tab content, using existing `Card`-style classes. Their dismissal timestamps are stored under their own `localStorage` keys (`mi_nomina_install_hint_dismissed`, `mi_nomina_backup_snooze`). iOS detection: `/iPhone|iPad|iPod/.test(navigator.userAgent) || (navigator.platform === 'MacIntel' && navigator.maxTouchPoints > 1)`; standalone detection: `navigator.standalone || matchMedia('(display-mode: standalone)').matches`.

### 10. Hosting on GitHub Pages
Push the repo to GitHub (private repo + Pages requires a paid plan; a public repo is fine because the repo holds only code and demo data. Real data stays in the browser, and `*.json` exports are git-ignored). Serve from the default branch root. All paths in the manifest and service worker are relative, so a project-pages subpath (`/mi-nomina/`) works.
*Alternatives:* Netlify / Cloudflare Pages drag-and-drop (also fine, no repo needed). The design only assumes "static HTTPS host with relative paths".

### 11. Icons
A simple generated icon (white "N" or peso/calendar glyph on indigo `#4338ca`), exported as PNG at 180 (apple-touch-icon), 192, 512, and 512 maskable with safe-zone padding, stored in `icons/`. iOS ignores SVG touch icons, so PNG is required.

## Risks / Trade-offs

- **Separate storage between Safari and the installed app** → Users will think their data vanished after installing. Mitigation: the install hint and README explain "Exportar en el computador → Importar en el iPhone". On first launch in standalone mode with only demo data, the dashboard shows an "Importar datos" call to action.
- **Data loss if the app is deleted or storage is evicted** → `navigator.storage.persist()` and the 30-day backup reminder. Exports are the real safeguard; there is no server copy by design.
- **Two devices diverge** (edits on phone and desktop) → Import is a full replace, not a merge. The confirm dialog states this. Recommended workflow in the README: pick one primary device and treat the other as read-only, or always export before switching. Merge is out of scope.
- **Stale cached app after deploy** → Versioned cache plus update prompt. Developer must bump `CACHE_VERSION` on every deploy. Mitigation: documented in CLAUDE.md's update checklist; the version string sits next to the yearly constants.
- **Babel Standalone size (~3 MB) on first load over cellular** → One-time cost, cached afterwards. Acceptable for a personal tool.
- **Payroll constants change in January but the phone keeps running old cached code** → Same mitigation as stale cache: the January update is a deploy with a bumped cache version, and the user sees the update prompt.
- **iOS Web Share quirks** (older iOS without file sharing) → Download fallback. iOS 15+ is assumed, which covers all supported iPhones.

## Migration Plan

1. Implement and test locally with `python -m http.server` (`localhost` counts as a secure context, so the service worker runs) and in Safari's Responsive Design Mode / iOS Simulator.
2. Create the GitHub repo, push, and enable Pages.
3. On the Mac: open the old `index.html` (`file://`) → Exportar.
4. On the iPhone: open the Pages URL in Safari → Add to Home Screen → launch → Importar (AirDrop or Files the export).
5. Rollback: hosting is additive. The local `file://` workflow keeps working throughout. To roll back a bad deploy, revert and push with a bumped `CACHE_VERSION`. To kill the service worker entirely, deploy an `sw.js` that unregisters itself and clears caches.

## Open Questions

- GitHub Pages (public repo) vs. Netlify/Cloudflare drop: decide at deploy time. Nothing in the implementation depends on it.
- Icon artwork: placeholder glyph now, or a designed icon?
- Should the desktop `file://` version show a "use the hosted URL instead" nudge, so there's one canonical place? Not in scope for now.

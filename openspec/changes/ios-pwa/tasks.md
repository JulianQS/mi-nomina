## 1. Vendor runtime libraries

- [x] 1.1 Download React 18.3.1 and ReactDOM 18.3.1 production UMD builds and a pinned Babel Standalone 7.x into `vendor/`, with versions in the filenames or headers
- [x] 1.2 Replace the three unpkg `<script>` tags in `index.html` with local `vendor/` paths; verify the app renders under `file://` and `python -m http.server`
- [x] 1.3 Confirm in the Network panel that the app makes no third-party requests

## 2. Installability

- [x] 2.1 Create app icons in `icons/`: `apple-touch-icon-180.png`, `icon-192.png`, `icon-512.png`, `icon-512-maskable.png` (indigo `#4338ca` background)
- [x] 2.2 Add `manifest.webmanifest` (name, short_name, lang, relative start_url/scope, standalone, theme/background colors, icons) and confirm `git status` shows it as untracked (not ignored)
- [x] 2.3 Add head tags to `index.html`: manifest link, apple-touch-icon, `apple-mobile-web-app-title`, `apple-mobile-web-app-capable`, `mobile-web-app-capable`, `apple-mobile-web-app-status-bar-style=black-translucent`, `theme-color`, and `viewport-fit=cover` on the viewport meta

## 3. Offline support

- [x] 3.1 Write `sw.js`: `CACHE_VERSION`, precache list (index.html, manifest, icons, vendor files), cache-first same-origin GET handler, delete old caches on activate, `SKIP_WAITING` message handler
- [x] 3.2 Add a plain `<script>` in `index.html` that registers `sw.js` only when `serviceWorker` is supported and the protocol is http(s); confirm `file://` shows no errors
- [x] 3.3 Add the "Nueva versión disponible — recargar" prompt: detect a waiting worker, post `SKIP_WAITING`, reload on `controllerchange`
- [x] 3.4 Test offline: load via localhost, go offline in DevTools, reload; all tabs work and data persists. Bump `CACHE_VERSION` and confirm the update prompt appears

## 4. Mobile layout

- [x] 4.1 Add safe-area CSS (`.safe-top`, `.safe-bottom`) to the plain `<style>` block and apply it to the header, the bottom tab bar, and the `BottomSheet` content; increase the main content bottom padding to cover the tab bar plus inset
- [x] 4.2 Add `@media (pointer: coarse)` rule forcing inputs/selects/textareas to 16px font size; verify that focusing a field in iOS Simulator doesn't zoom
- [x] 4.3 Add `-webkit-tap-highlight-color: transparent` and `overscroll-behavior-y: none` (or matching html background) to avoid white overscroll in standalone
- [x] 4.4 Make tab bar buttons, Importar/Exportar, mark-as-paid and sheet close buttons at least 44×44 px (use classes already in the Tailwind subset or plain CSS)

## 5. Data portability

- [x] 5.1 Extract the `pto_db` migration from `loadDB` into `migrateDB(obj)` and add `validateDB(obj)` (employees array; each has id, name, salary_history array) returning an error reason or null
- [x] 5.2 Rewrite `importJSON`: parse → migrate → validate → `confirm` with employee count → `setDb`, fixing `active` if it doesn't match; show "Archivo inválido: <reason>" on failure; set `accept="application/json,.json,text/plain"`
- [x] 5.3 Rewrite `exportJSON`: dated filename, `navigator.share({ files })` when `canShare` allows, ignore `AbortError`, download fallback; write `mi_nomina_last_export` on success
- [x] 5.4 Request `navigator.storage.persist()` once at startup, guarded and silent on failure
- [x] 5.5 Add the backup reminder banner (no export or older than 30 days; dismiss snoozes 7 days via `mi_nomina_backup_snooze`) with an "Exportar ahora" action
- [x] 5.6 Add the first-run "Importar datos" prompt shown in standalone mode while the DB equals the demo data

## 6. Install hint

- [x] 6.1 Add `isIOS()` / `isStandalone()` helpers and a dismissible "Instalar: Compartir → Agregar a inicio" banner shown only in iOS Safari (not standalone), persisted via `mi_nomina_install_hint_dismissed`

## 7. Docs and deploy

- [x] 7.1 Update README: hosted URL, iPhone install steps, desktop→iPhone data transfer, note that Safari and the installed app have separate storage, backup advice
- [x] 7.2 Update CLAUDE.md: new files (`sw.js`, `manifest.webmanifest`, `vendor/`, `icons/`), "bump `CACHE_VERSION` on every deploy" in the yearly-update section, and new CSS going in the plain `<style>` block
- [ ] 7.3 Create the GitHub repo, push, enable Pages, and confirm the hosted URL loads over HTTPS with the service worker active
- [ ] 7.4 On a real iPhone: Add to Home Screen, verify icon/name/full-screen launch, safe areas, no input zoom, airplane-mode launch, export via share sheet to Files, and import from Files

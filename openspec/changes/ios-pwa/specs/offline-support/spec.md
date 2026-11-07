## ADDED Requirements

### Requirement: Vendored runtime libraries
The app SHALL load React 18, ReactDOM 18 and Babel Standalone from local files under `vendor/`, pinned to exact versions and using production builds for React/ReactDOM. The app SHALL NOT depend on any third-party host at runtime.

#### Scenario: No external requests
- **WHEN** the app loads
- **THEN** every script and stylesheet is requested from the app's own origin (or from disk under `file://`)

#### Scenario: Pinned versions recorded
- **WHEN** a developer inspects `vendor/`
- **THEN** each vendored file's name or header states its exact version

### Requirement: App shell caching
When served over HTTPS (or `localhost`), the app SHALL register a service worker (`sw.js`) that precaches `index.html`, `manifest.webmanifest`, the icons and the vendored libraries, and serves them cache-first so the app opens and functions fully without network access.

#### Scenario: Offline launch
- **WHEN** the app has been opened once online and the device is then put in airplane mode
- **THEN** launching the app from the home screen loads it fully, with all tabs and calculations working and existing data visible

#### Scenario: Data changes offline
- **WHEN** the user records an absence or marks an obligation paid while offline
- **THEN** the change is saved to `localStorage` exactly as when online

### Requirement: Versioned updates
The service worker SHALL use a cache name that includes an explicit version string. When a new version is deployed, the app SHALL install the new cache, delete old caches on activation, and use the new version no later than the next launch after the update was fetched.

#### Scenario: New version deployed
- **WHEN** `sw.js` is deployed with a bumped version and the user opens the app online
- **THEN** the new files are cached in the background and, on the next launch, the new version runs and old caches are removed

#### Scenario: Update notice
- **WHEN** a new version has been installed while the app is open
- **THEN** the app shows a non-blocking "Nueva versión disponible — recargar" prompt, and tapping it reloads into the new version

### Requirement: Local file mode still works
Opening `index.html` directly from disk (`file://`) SHALL continue to work as today. Service worker registration SHALL be skipped when the protocol is not `http:`/`https:` or the context is not secure, without errors that break the app.

#### Scenario: Open from Finder
- **WHEN** a user double-clicks `index.html` on macOS
- **THEN** the app renders and works, and no service worker registration is attempted

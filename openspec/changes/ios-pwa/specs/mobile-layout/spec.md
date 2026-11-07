## ADDED Requirements

### Requirement: Safe-area insets
The viewport SHALL use `viewport-fit=cover`, and the layout SHALL pad the header by `env(safe-area-inset-top)` and the bottom tab bar by `env(safe-area-inset-bottom)`, so no content is hidden behind the notch, Dynamic Island, status bar or home indicator. Content scrolling under the tab bar SHALL keep enough bottom padding to remain reachable.

#### Scenario: Standalone on a notched iPhone
- **WHEN** the installed app runs on an iPhone with a notch or Dynamic Island
- **THEN** the employee name and header buttons sit below the status bar, and the tab labels sit above the home indicator

#### Scenario: Last item reachable
- **WHEN** the user scrolls to the bottom of the longest view
- **THEN** the last element is fully visible above the tab bar

#### Scenario: Bottom sheets
- **WHEN** a bottom sheet (e.g. employee picker) is open on an iPhone with a home indicator
- **THEN** its content is padded so the last control is above the home indicator

### Requirement: No input auto-zoom
Text inputs, number inputs, date inputs and selects SHALL render with a font size of at least 16px on touch devices, so iOS Safari does not zoom the page on focus. User pinch-zoom SHALL NOT be disabled.

#### Scenario: Focus an input
- **WHEN** the user taps a form field on an iPhone
- **THEN** the keyboard opens and the page does not zoom in

### Requirement: Touch-friendly controls
Interactive controls (tab bar buttons, header Importar/Exportar, mark-as-paid actions, bottom-sheet close button) SHALL have a tap target of at least 44×44 CSS px, and tapping them SHALL NOT show the default grey tap highlight.

#### Scenario: Tap targets
- **WHEN** the app is measured at 390px viewport width
- **THEN** each listed control's hit area is at least 44×44 px

### Requirement: Standalone appearance
In standalone mode the status bar area SHALL show the header's indigo background (status bar style "black-translucent" with the header extending under it), and the page SHALL NOT rubber-band to reveal a mismatched background color at the top.

#### Scenario: Pull down at top
- **WHEN** the user overscrolls at the top of the app in standalone mode
- **THEN** the revealed area matches the header color or the page background, not white

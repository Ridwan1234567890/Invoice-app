# Invoice App

A perfect, fully responsive Invoice Management Application built from Figma designs using React (CDN), vanilla CSS, and localStorage — delivered as a single HTML file with zero build tools.

-----

## Setup

No installation required. Open `invoice-app.html` directly in any modern browser, or serve it locally:


**Requirements:** Chrome 90+ / Firefox 88+ / Safari 14+ / Edge 90+. An internet connection is needed on first load for Google Fonts and the React CDN. To reset data to defaults, run `localStorage.removeItem('inv3')` in the browser console and refresh.

-----

## Architecture

The entire app lives in one `.html` file, structured in three layers:

```
invoice-app.html
├── <style>       Design tokens, layout, components, responsive breakpoints
└── <script>      Seed data · Utils · Icons · Badge · EmptyState · Form · App
```

**Routing** is handled by a single `view` state variable (`'list' | 'detail'`) — no React Router needed for a two-screen app. **Theming** uses CSS custom properties; dark mode is a single attribute toggle (`data-dark="1"` on `<html>`), so every component re-themes automatically with zero JavaScript per element. **Persistence** is a `useEffect` that serialises the invoices array to localStorage on every change, with lazy initial state reading it back on mount.

```
App (state owner)
 ├── invs[]        All invoices — source of truth
 ├── view          'list' | 'detail'
 ├── dark          Boolean — persisted to localStorage
 ├── selId         Currently viewed invoice
 ├── drawerOpen    Form drawer visibility
 ├── editId        null = new invoice · string = edit mode
 └── filters[]     Active status filters
```

All state lives in `App`. `Form` owns its own local field state and communicates upward via an `onSave(invoice)` callback. No external state library is needed at this scale.

-----

## Trade-offs

|Decision                 |Reason                                                        |Downside                                                           |
|-------------------------|--------------------------------------------------------------|-------------------------------------------------------------------|
|Single HTML file         |Zero setup, maximum portability                               |No HMR, no tree-shaking, Babel runs in-browser (~1s cold parse)    |
|View-state routing       |No dependency overhead                                        |Browser back button doesn’t navigate between views; no deep linking|
|localStorage             |No server required, survives refresh                          |Per-browser only, no multi-device sync, single-tab                 |
|Inline SVG icons         |Exact Figma match, themeable via `currentColor`, no dependency|More verbose than an icon library                                  |
|CSS variables for theming|Zero runtime cost, native browser support                     |Less composable than CSS-in-JS at larger scale                     |

-----

## Accessibility

- **Semantic HTML** — `<nav>`, `<header>`, `<article>`, `<section>`, `<address>`, `<footer>` landmarks throughout. All inputs have paired `<label>` elements.
- **Keyboard navigation** — all interactive elements are Tab-reachable. Invoice rows respond to `Enter` and `Space`. ESC closes the drawer or modal from any focused element inside.
- **Focus trap** — the delete modal traps Tab/Shift+Tab within its two buttons and moves focus to Cancel on open.
- **ARIA** — drawer uses `role="dialog" aria-modal="true"`; delete modal adds `aria-labelledby`; filter uses `role="listbox" aria-multiselectable`; badges use `role="status"`; toggle button `aria-label` updates with state.
- **Colour contrast** — primary text pairs meet WCAG AAA. The muted label colour (`#888EB0`) and status badge colours are borderline AA for small text — an intentional carry-over from the Figma design system.

-----

## Improvements

**High priority**

- **Hash-based routing** — replace view-state switching with `window.location.hash` so the browser back button works and invoices are deep-linkable (`/#/invoice/XM9141`).
- **Focus return on modal close** — after the delete modal closes, focus should return to the Delete button that triggered it (requires storing a ref to the trigger).

**Medium priority**

- **Build pipeline** — migrate to Vite + React + TypeScript for HMR, tree-shaking, type safety, and proper module imports, removing the in-browser Babel cost.
- **Backend / IndexedDB** — replace localStorage with a REST API or IndexedDB for multi-device sync, larger storage, and real error handling.
- **Tests** — unit tests for `validate()`, `calcTotal()`, and `buildInv()`; integration tests for the full CRUD flow with React Testing Library.

**Nice to have**

- Undo delete (toast with 5-second window before committing)
- Invoice PDF export via `html2canvas` or `jsPDF`
- Real-time search by client name, ID, or description
- `@media print` styles for browser-native printing

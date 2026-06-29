# KSRTC Prototype — Claude Context

## What this project is
Single-file HTML prototype for KSRTC's OPRS (Online Passenger Reservation System) booking flow.
File: `ksrtc-prototype.html` — open directly in Chrome.
No build step. All CSS and JS is inline in the one file.

## Owner
Amarpreet Kaur — Product / UX Designer, 8 years experience. Strong in UI/UX detail.
Adjust depth accordingly: skip basics, give context on non-obvious technical decisions.

## Related project
The OPRS Design System lives at: `ksrtc-design-system/`
That project has its own CLAUDE.md. Work on DS components in that folder, not here.

## Tech stack
- Vanilla HTML/CSS/JS — no framework
- ag-Grid Community v27+ for the search results data table
- Material Symbols (Google Fonts icon font)
- CSS custom properties for tokens (see token reference below)

## Token reference (prototype uses DIFFERENT names than the DS)
```
--primary: #1A56A0              (blue)
--secondary: #D84315            (deep orange)
--secondary-container: #FFDBCA
--primary-container: #D6E4FF
--outline: (border)
--outline-variant: #CAC4D0
--surface: #FFFFFF
--surface-1: #EEF2F9
--on-surface: #1C1B1F
--on-surface-variant: (muted text)
```
Note: DS tokens use `--color-primary`, `--color-secondary` etc. Prototype drops the `--color-` prefix.

## Button classes (prototype)
| Class | Visual | Use |
|---|---|---|
| `.btn` + `.btn-filled` | Orange fill | Primary CTA |
| `.btn-outlined` | Orange border, transparent | Secondary |
| `.btn-tonal` | Orange border, transparent | Same as outlined here |
| `.btn-orange-sm` | Small orange fill | Compact actions |
| `.icon-btn-outlined` | Blue border | Icon-only (swap) |

## ag-Grid seat panel — critical knowledge
The "Select Seats" panel opens as a **full-width ag-Grid row** inserted after the selected bus row.

### How it works
1. User clicks "Select" → `selectBus(idx)` is called
2. `_detailRowHeight = _calcDetailHeight(busData.preSelected.length)` is calculated FIRST
3. `_insertDetailRow(gridId, idx)` inserts `{ _detail: true }` row via `api.setRowData(rows)`
4. `getRowHeight` reads `_detailRowHeight` and returns it for the detail row
5. `fullWidthCellRenderer` creates a div and moves `#seat-panel-wrap` into it via setTimeout(0)
6. Auto-scroll fires at 60ms to bring the panel into view

### Height formula
```js
var _DETAIL_BASE_H = 830;   // fixed sections (seat map + boarding + pax form + fare + action bar)
var _DETAIL_PAX_H  = 44;    // per passenger row height
// 1 seat = 874px, 2 = 918px, 3 = 962px
```

### HARD RULES — do not violate these (caused major breakage before)
1. **NEVER call `api.resetRowHeights()`** while the seat panel is open. It destroys the full-width row DOM and wipes the seat panel.
2. **NEVER add `position: relative; z-index` to `.ag-full-width-container` or `.ag-full-width-row`**. It breaks ag-Grid's internal absolute-positioning layout — group headers misalign, rows overlap.
3. **Height is calculated once on open** — not updated live when seats are toggled. Removing the live-resize was intentional (it triggered `resetRowHeights` which caused rule #1 to fire).

## Seat panel HTML structure
```
#seat-panel-wrap.ag-seat-panel-wrap   ← display:none / .open = display:block
  #seat-panel
    .seat-panel-grid (1fr 1fr)
      .seat-map-col   ← bus layout, legend, seat selection
      .boarding-col   ← boarding/dropping points, pax form, fare card, action buttons
```

## Layout rules
- `.seat-panel-grid`: `grid-template-columns: 1fr 1fr` (equal halves — do not change)
- `.seat-map-col`: `display:flex; flex-direction:column; align-items:center` (centered bus)
- `.seat-map-title`: `align-self: flex-start` (title stays left-aligned within centered col)
- `.fare-row`: `grid-template-columns: 1fr 1fr` (ticket stub + fare card, equal)

## Screen flow
1. **Search screen** (`#screen-search`) — route + date search form + results grid
2. **Ticket screen** (`#screen-ticket`) — booking confirmation

### Key functions
| Function | Purpose |
|---|---|
| `selectBus(idx)` | Opens seat panel for onward journey row |
| `selectReturnBus(idx)` | Opens seat panel for return journey row |
| `closeSeatPanel()` | Removes detail row, rescues seat-panel-wrap DOM first |
| `toggleSeat(n)` | Toggles seat selection, updates pax rows + fare card |
| `addPaxRow(n)` | Adds a passenger form row for seat n |
| `updateFareCard()` | Recalculates and renders fare summary |
| `_insertDetailRow(gridId, busIdx)` | Inserts `_detail` row into ag-Grid |
| `_removeDetailRow(gridId)` | Removes `_detail` row safely |
| `buildGrid(id, seats)` | Renders seat cells into upper/lower berth containers |
| `generateSeatData(count, seed)` | Generates mock seat availability data |

## Known issues / watch list
- `_removeDetailRow` must rescue `#seat-panel-wrap` BEFORE calling `api.setRowData` — otherwise ag-Grid destroys the DOM node.
- `selectBus` and `selectReturnBus` both set `_detailRowHeight` — if you add a third grid, add the same line before `_insertDetailRow`.
- The scroll-into-view uses two methods: `api.ensureIndexVisible` (ag-Grid internal) + `wrap.scrollIntoView` (window level). Both needed because `.grid-scroll-wrap` has `overflow: visible`.

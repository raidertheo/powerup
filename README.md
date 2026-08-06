# ⏱ TimeTrack — Trello Power-Up

A full-featured time-tracking, team-attribution, and showcase-image Power-Up for Trello, with a board-wide summary dashboard and Excel export.

---

## Features

### Per Card — "TimeTrack" popup (4 tabs)

| Tab | What it does |
|---|---|
| **Time** | Estimated hours, auto-calculated hours spent (from Log Hours), 0–100% progress slider, auto-save, over-budget warning, **P4V Integrated** checkbox |
| **Log Hours** | Log your own hours with an optional note; running history with delete; totals |
| **Assign** | Toggle which board members are attributed to this card; auto-syncs with Trello's native card Members (checks/unchecks automatically as members are added/removed on the card) |
| **Showcase** | Pick an image from the card's existing Trello attachments (full resolution, shown to every teammate automatically), or manually upload/compress a thumbnail as a fallback |

Badges on the card front and back show at-a-glance status (time, team log, showcase) and open directly to the relevant tab.

### Board Time Summary (via the "Time Summary" board button)

| Tab | What it shows |
|---|---|
| **Chart** | Pie chart — pick "Board" or any individual list, colored by list |
| **Label** | Progress % per Trello label, matched to your board's real label colors |
| **List** | Color-coded progress bars per list, customizable colors |
| **Member** | Hours logged vs. estimated per person; click through to a drill-down of every assigned card grouped by list, with live status (**PENDING** / **IN PROGRESS** / **COMPLETE**) and over/under-budget coloring (green = on/under budget, yellow = in progress, red = over budget) |
| **⚙ Settings** | Assign custom colors to lists and to team members (shared 10-color palette: Green, Navy, Purple, Gray, Orange, Red, Blue, Yellow, Teal, Pink) |

### Export (via "Export to Excel" board button)

- **Download Excel (.xlsx)** — Type, Item, Item Count, Phase, Est Hours, Worked Hours, Status (Pending/In Progress/Complete), Integrated (Yes/blank), Resource, Notes — with a totals row, center-aligned Item Count, and rows sorted alphabetically by list then card name
- **Download with Images (.xlsx)** — same data plus each card's Showcase image embedded directly in the spreadsheet, with uniform column widths and text wrapping
- **Add Missing Images** — for cards with a Showcase image chosen but no image data cached yet: shows each card's suggested filename (e.g. `Card_Name.jpg`), an **"Open All"** button to pop every attachment open in one click, then a single folder picker that reads all your saved files back and **automatically matches them to cards by filename** (with manual pairing as a fallback for anything that doesn't auto-match)

---

## Files

```
trello-powerup/
├── index.html          ← Power-Up connector — REQUIRED
│                          (board buttons, card badges/buttons, builds Time Summary + Export data)
├── card-popup.html      ← REQUIRED — the 4-tab TimeTrack popup opened from every card
├── list-summary.html    ← REQUIRED — the Time Summary board popup
├── export.html          ← REQUIRED — the Export to Excel popup
└── README.md
```

> Older files (`popup.html`, `team-log.html`, `auth.html`) are **legacy from earlier iterations** and are no longer used — you can safely leave them out of your deployment. Only the four files above are needed.

---

## How the data works (good to know before troubleshooting)

- **Per-card data** (Time, Log Hours, Assign, Showcase selection) is stored using Trello's own card-scoped Power-Up storage (`t.set`/`t.get`), so it syncs across every device and team member automatically.
- **Board Time Summary and Export data** are built by the connector (`index.html`) — which has full, reliable access to every card — and then handed off to the popups via the browser's `localStorage` (since all files share the same hosting origin). This sidesteps Trello's small per-key storage limit, which is easy to hit once summary/export data covers a whole board.
- **List and Member colors** are stored via Trello's *shared* board storage, so the whole team sees the same custom colors.
- Because of the `localStorage` handoff, **Time Summary and Export need to be opened at least once after any major board changes** to rebuild their cached data — just click the board button again.
- **Showcase images shown via a native Trello attachment** are full resolution and visible to everyone automatically — no size limits, since it's a real Trello attachment, not data we store ourselves.

---

## Setup Instructions

### Step 1 — Host the files

Needs HTTPS. Options:

- **[Glitch.com](https://glitch.com)** — free, instant, no config
- **[Vercel](https://vercel.com)** — free tier, `vercel deploy`
- **[Netlify](https://netlify.com)** — drag-and-drop the folder
- **GitHub Pages** — push to a repo, enable Pages

### Step 2 — Create the Power-Up on Trello

1. Go to **[https://trello.com/power-ups/admin](https://trello.com/power-ups/admin)**
2. Click **"New Power-Up"**
3. Fill in:
   - **Name**: TimeTrack
   - **Connector URL**: `https://your-hosted-url/index.html`
4. Under **Capabilities**, enable:
   - `card-badges`
   - `card-detail-badges`
   - `card-buttons`
   - `board-buttons`
5. Copy the **API Key** shown on the page
6. Open `index.html`, `card-popup.html`, `list-summary.html`, and `export.html`, and replace `YOUR_TRELLO_API_KEY` with your key in **all four files**

### Step 3 — Add to your board

1. Open any Trello board
2. Click **Power-Ups** in the board menu
3. Search for **"TimeTrack"** (or Custom) and click **Add**

---

## Using it

### On a card
1. Click **TimeTrack**
2. **Time** — set estimated hours, drag the progress slider (auto-saves), check **P4V Integrated** if applicable
3. **Log Hours** — log your own hours against the card
4. **Assign** — pick who's attributed to this card (auto-fills from Trello's Members field)
5. **Showcase** — attach an image to the card the normal Trello way, then pick it from the list in this tab (full resolution, visible to everyone); or use "Choose Image" to upload a compressed thumbnail directly

### Board summary
- Click **Time Summary** in the board toolbar
- Browse Chart, Label, List, and Member tabs
- Click any member to drill into their assigned cards
- Use **⚙ Settings** to customize list and member colors

### Export
- Click **Export to Excel** in the board toolbar
- Choose plain or image-inclusive `.xlsx` download
- If some cards are missing cached images for the image export, use **"Add Missing Images"** to save and re-match them by filename

---

## Known limitations

- **Showcase thumbnails uploaded via "Choose Image"** are small (~100px) — Trello's per-card storage budget is shared across all of TimeTrack's data on that card (time, log history, assignments, image), so images are aggressively compressed to leave room for everything else. Using a real Trello attachment instead avoids this entirely (full resolution, no compression).
- **Cards with very long Log Hours history** could eventually approach that same storage limit — if you hit a save failure on a heavily-logged card, that's why.
- **The "Cache High-Res Copy" button** (Showcase tab) is experimental — Trello's attachment servers typically block the browser security check it needs (CORS), so it usually falls back to the compressed thumbnail. This is a platform limitation, not a bug.
- **The Excel "with images" export** can't automatically read bytes from Trello-hosted attachments for the same CORS reason — the "Add Missing Images" flow works around this by having you save the files locally (with a suggested filename) and then select the folder back, matching by filename.
- The `localStorage` handoff for Time Summary/Export means that data doesn't sync live across devices — reopen the relevant board button on each device to refresh it.

---

## Customization

| What | Where |
|---|---|
| Colors / theme | CSS `:root` variables in each HTML file |
| List/Member color palette options | `SWATCHES` / `MEMBER_SWATCHES` arrays in `list-summary.html` |
| Showcase thumbnail size/quality (manual upload) | `SHOWCASE_MAX_DIM` / `SHOWCASE_MAX_BYTES` in `card-popup.html` |
| Excel export image size | `IMAGE_HEIGHT` / `IMAGE_ROW_HEIGHT` in `export.html` |
| "Add Missing Images" compression | `PROVIDE_MAX_DIM` / `PROVIDE_QUALITY` in `export.html` |

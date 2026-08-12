# Lake Side Detailing — Live Sales Leaderboard

`sales-leaderboard.html` — double-click it. It opens in your browser and works offline.

Right now it's showing **demo data** so you can see the layout. Two things to fix to make it real.

---

## ⚠️ Tracker v2 — read this first

`Lake_Side_Detailing_Sales_Tracker_v2.xlsx` has **seven tabs**, and the one your form
writes into is **not** the one the workbook's own dashboard reads.

| Tab | What it is |
|---|---|
| `Form1` | **The live Microsoft Forms responses.** This is the real data. |
| `DEALS` | A manual deal log. Header sits on **row 5**. Nothing feeds it automatically. |
| `DASHBOARD` `REPS` `WEEKLY` `LOOKUP` | Formulas — all read `DEALS`, never `Form1`. |
| `SETUP` | Roster + season goal. |

**The two are not connected.** Their columns don't even match:

- `Form1` → `ID, Start time, Completion time, Email, Name, Customer Name, Address, Phone, Email1, Vehicle Type, Price, Scheduled Date, IF NOT SCHEDULED — liklihood`
- `DEALS` → `Timestamp, Rep, Customer Name, Service Address, Service Description, Price, Service Date, Notes, New/Repeat`

So the workbook's DASHBOARD showing **$710 · 3 deals · Top Rep "Jake M."** is computed
entirely from the **three example rows** the template shipped with (Dana Reilly $285,
Marcus Webb $140, D. Reilly $285). Those are fake. Meanwhile your one real submission —
**Larry Gibas, Trailer, $999, scheduled Aug 7** — sits in `Form1` and is invisible to
every number on that dashboard.

The template was also written for a **Google** Form ("send the Google Form link to your
reps"), not Microsoft Forms. And the `SETUP` roster still lists the template's example
reps: `Jake M.`, `Tyler R.`, `Sam K.`

**This HTML board reads `Form1` directly**, so it is unaffected by that break. It scans
every tab, scores them, and picks the live form-response tab automatically (it also
finds headers that aren't on row 1). Override with `CONFIG.sheetName` if it ever guesses
wrong. The footer tells you which tab it chose.

**Also delete `Form1` row 2** — the `csdmjsd n / dsksdcnk` test row. Its price is text,
so it counts as a $0 lead.

## Fix #1 — Add a "Sales Rep" field to your form (required)

⚠️ **v2 still has no rep field, and switching to "Anyone can respond" made this worse.**
The form page now states it "will not automatically collect your details like name and
email address." While the form was org-restricted it auto-filled `Name` and `Email`
(that's why the Larry Gibas row says *Charlie Willsey*) — **those columns will be blank
from now on.** Every deal your reps submit is anonymous and can never be attributed
after the fact. Add this before they start logging.

Your v1 form collected: `Full name, Address, Email, Phone number, City, ZIP code,
Estimated total cost, Any special requests?`

None of those say **who on the team got the deal** — so there is nothing to rank. Add it:

1. Open the form (the one behind `QRCode for Lake_Side_Detailing_Sales_Tracker.png`).
2. **+ Add new** → **Choice**.
3. Title it exactly `Sales Rep`. Add these five options, **exactly as written**:
   - Jack Murphy
   - Will Hanley
   - Danny Dempsey
   - Winston Vassilos
   - Danny Cosgrove
4. Toggle **Required** on.

> ⚠️ **Two Dannys — use full names in the form options.** If a submission ever comes
> through as just `Danny`, the board treats it as a phantom sixth rep and neither
> Dempsey nor Cosgrove gets the credit. Never use a free-text field for this; the
> Choice question is what prevents typos from splitting someone's total across
> "Winston V", "winston", and "Winston Vassilos".

Optional but recommended — a second Choice field titled `Deal Status` with options
`Won` / `Lost` / `Pending`. Without it, every submission counts as a won deal and
close rate can't be calculated.

The board auto-detects both columns by name. You don't have to touch the HTML.

> ⚠️ Existing submissions won't have a rep. They'll show as `Unassigned` until you
> backfill that column in the sheet, or just ignore them and start clean.

---

## Fix #2 — Give it a live feed

The board needs a URL it can re-pull on a timer. A downloaded `.xlsx` in your
Downloads folder is a dead snapshot — nothing can push updates into it.

### The important catch

Your form is **Microsoft Forms**, not Google Forms — I can tell from the
`Start time / Completion time / Email / Name` columns and the teal QR code.
Microsoft doesn't offer a no-auth public CSV link the way Google does, so pick one:

### Option A — Move the funnel to Google Forms (simplest, fully live)

1. Rebuild the same questions at [forms.google.com](https://forms.google.com) — add
   `Sales Rep` and `Deal Status` this time.
2. **Responses** tab → green Sheets icon → creates a linked response spreadsheet.
3. In that sheet: **File → Share → Publish to web**.
4. Left dropdown: pick the responses sheet. Right dropdown: **Comma-separated values (.csv)**.
5. **Publish** → copy the URL. It looks like:
   `https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=0&single=true&output=csv`
6. Open `sales-leaderboard.html` in a text editor, find `csvUrl: ""` near the top,
   paste it between the quotes. Save. Reload.
7. Re-print the QR code from the new Google Form.

Done — the board now refreshes itself every 30 seconds, forever.

### Option B — Keep Microsoft Forms, bridge into Google Sheets

Keep your existing form and QR code. In a Google Sheet, use `IMPORTRANGE`/`IMPORTDATA`
against the OneDrive workbook, or add a Power Automate flow ("When a new response is
submitted" → "Add row" into a Google Sheet). Then publish *that* sheet as CSV per
steps 3–6 above. More moving parts, but the printed QR code stays valid.

### Option C — Manual, no setup

Skip `csvUrl` entirely. Download the tracker from Forms whenever you want an update
and drag the `.xlsx` straight onto the page. Reads it instantly, no internet needed.
Fine for a weekly team meeting; not a wall display.

**Publishing note:** "Publish to web" makes that sheet readable by anyone with the
link. Only publish a tab holding the columns you want on a leaderboard —
rep, deal value, status, date. Don't publish customer addresses, emails, and phone
numbers. Easiest way: a second tab that pulls just those columns, and publish only it.

---

## Settings

Open the file in a text editor. Everything tunable is in the `CONFIG` block at the top:

| Setting | What it does |
|---|---|
| `csvUrl` | The live feed. Empty = demo/manual mode. |
| `refreshSeconds` | How often to re-pull. Default 30. |
| `roster` | Your five salesmen. Everyone here shows even at zero deals. |
| `awards.hustleMetric` | `bestDay` / `volume` / `biggest` — see Recognition below. |
| `awards.rookieWindowDays` | How recent a rep's first deal must be. Default 30. |
| `awards.minDecidedForCloseRate` | Min decided deals to qualify. Default 3. |
| `monthlyDealGoal` | Team target — drives the "% of goal" readout. |
| `monthlyRevenueGoal` | Same, in dollars. |
| `columns` | Force a specific column if auto-detect guesses wrong. Use the exact header text. |
| `wonValues` / `lostValues` | Which status words count as won or lost. |

## On the page

- **This Month / Week / Today / All Time** — time filters
- **Load .xlsx** — or just drag the file anywhere onto the page
- **Demo Data** — reload the sample set
- **Fullscreen** — for a TV in the shop

When a new deal lands, it flashes in the Latest Deals list and pops a toast in the corner.

## Recognition — the Instagram cards

The **Recognition** panel picks a winner in five categories. Click any one and it
downloads a 1080×1080 PNG, branded and ready to post. Nothing is uploaded anywhere —
the image is drawn in the browser and saved straight to your Downloads folder.

| Category | How the winner is chosen |
|---|---|
| 🏆 Top Closer | Most deals in the selected period; revenue breaks ties. |
| 🌟 Rookie of the Week | Most deals among reps whose **first-ever** deal was in the last 30 days. |
| 📈 Most Improved | Biggest gain vs. the previous equal period. Blank on "All Time" — nothing to compare against. |
| 🔥 Biggest Hustle | **Assumption: most deals closed in a single day.** Change `awards.hustleMetric` to `volume` (most opportunities worked) or `biggest` (largest single deal) if you'd rather reward something else. |
| 🎯 Highest Close Rate | Won ÷ decided. Needs a `Deal Status` column, and ignores anyone under 3 decided deals so a 1-for-1 can't win. |

A greyed-out row means nobody qualified yet — it tells you which condition wasn't met.
Cards follow whichever time filter is active, so the header reads "THIS WEEK" or
"THIS MONTH" to match.

**Worth knowing:** with only five reps, Top Closer and Rookie of the Week will often be
the same person early on, since everyone's first deal is recent. That resolves on its
own once the team has more than 30 days of history.

## Job Schedule (Google Calendar)

The **Job Schedule** panel embeds your *Lake Side Detailing* calendar
(`d28e2cc3…@group.calendar.google.com`). Agenda / Week / Month toggle, plus a button
that opens the real calendar in Google.

> ⚠️ **The embed code you sent contained four calendars** — your personal
> `charliewillsey@gmail.com`, your **Family** calendar, Lake Side Detailing, and US
> Holidays. Only Lake Side is wired in. This board is meant for a shop TV; do not paste
> the multi-src embed back in unless you want the crew reading your family's schedule.

### ⛔ It will look blank on a TV that isn't signed in

Your calendar is **private**. Checked 2026-08-05: requesting the embed while signed out
returns Google Calendar's *marketing page*, not your events. So the panel works on your
laptop (you're signed in) and shows nothing useful on a signed-out display.

Two ways to fix it — **pick the second one**:

1. **Make the calendar public** (Settings → *Access permissions* → Make available to
   public). Works everywhere instantly. ⚠️ But if event titles carry customer names,
   addresses, and prices, that data becomes public to anyone with the calendar ID.
   **Don't do this** with real customer bookings on it.
2. **Sign the shop display into a Google account that has the calendar shared to it**
   (read-only is enough). Customer data stays private and the panel renders. ✅

### Why the schedule can't feed the KPIs

Google's `.ics` endpoint sends **no `access-control-allow-origin` header** — verified
2026-08-05 against a known-public calendar. A browser page therefore cannot fetch
calendar data, so scheduled jobs can't roll into the funnel or the counters without a
server or a Sheets bridge. The panel is a live *view*, not a data source. If you want
booked-but-not-yet-closed jobs counted, the clean route is a Google Apps Script that
writes calendar events into the same published sheet the board already reads.

### Booking format

Book jobs as:

```
INITIALS: Customer — Service — $Price
```

For example: `JM: Monty Enoch — Ultimate Detail — $650`

Initials keep the agenda readable at a glance and match the leaderboard names.
**Two Dannys:** `DD` = Dempsey, `DC` = Cosgrove. This extends the `CW:` convention
already on the calendar.

Put the address in the event **location** field and the phone in the **description** —
not the title. That keeps the title short on the TV, and means that if you ever do make
the calendar public, the sensitive fields aren't in the headline.

## Putting it on a TV

You already have `WebViewScreenSaver` in Downloads — point it at the `file://` path of
`sales-leaderboard.html` and it'll run as a screensaver on any shop Mac. Or just open
it in a browser, hit **Fullscreen**, and disable sleep in Settings → Displays.

## Troubleshooting

**"Offline — Got HTML, not CSV"** — the link isn't a published-CSV link. It must end
in `output=csv`. A normal `/edit` browser URL won't work.

**Everyone shows as "Unassigned"** — no `Sales Rep` column yet. See Fix #1.

**Deals missing under This Month but visible under All Time** — the date column isn't
parsing. Check the footer: it names the columns it detected. Force it with
`columns.date` set to your exact header text.

**Revenue reads $0** — set `columns.value` to `"Estimated total cost"` explicitly.

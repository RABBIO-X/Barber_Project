# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A static booking front end for **Kettle Street Barbers**, a fictional three-chair barbershop. It's live on GitHub Pages at https://rabbio-x.github.io/Barber_Project/. Every push to `main` redeploys it within a minute or two.

There is **no backend yet**. Bookings exist only in the visitor's browser and disappear when the page reloads. The chosen direction for the backend is Square Appointments: one shop account with the barbers as team members, and one price per service. See `docs/backend-plan.md`.

## Layout

```
index.html        The whole app: one file with <style>, markup, then a <script> block (IIFE, no framework)
images/           Barber portraits (dee.jpg, marco.jpg, sam.jpg), 200x200, AI-generated in Canva
docs/             Architecture notes and backend plan
README.md         User-facing overview
```

There's no build step, package.json or dependencies. The only thing loaded from outside is Google Fonts (Big Shoulders Display, Public Sans, Caveat).

## Running and checking

- Run: open `index.html` in a browser, or `python -m http.server 8000`.
- Syntax-check the script after editing it (there's no test suite):
  ```sh
  sed -n '/^<script>$/,/^<\/script>$/p' index.html | sed '1d;$d' > /tmp/check.js && node --check /tmp/check.js
  ```
- After pushing, confirm the Pages build:
  `gh api repos/RABBIO-X/Barber_Project/pages/builds/latest --jq '.commit[0:7] + " " + .status'`

## Where things live in `index.html`

Data constants sit at the top of the `<script>` block:

| Constant | Meaning |
|---|---|
| `SERVICES` | `{ id, name, note, mins, price }`. One price per service, the same for every barber. This is a business decision, not a shortcut: don't add prices per barber. |
| `BARBERS` | `{ id, photo, name, short, tag, years, bio }`. Without `photo`, the barber's initials show. **Temporary stand-in:** profiles (photo, name, experience, working hours) will come from `GET /api/barbers`. See "Barber profiles come from the backend" in `docs/backend-plan.md`. |
| `HOURS` | Opening hours keyed by weekday (`0` = Sunday), as `[openMin, closeMin]` in minutes from midnight. `null` means closed (Monday). |
| `STEP` | Length of one slot in the times table, in minutes (30). |
| `BOOK_AHEAD_DAYS` | How far ahead people can book (14). The day picker shows today plus this many days. |

Key functions:

- `isTaken(day, barber, min)`: returns `"past"`, `"booked"` or `null`. Booked times are **fake**: they come from a fixed hash (the `busy` ratios per barber) plus the session's own bookings in `mine`.
- `canStart(day, barber, min)`: whether the chosen service fits in a row of free slots before closing.
- `findNext()`: the earliest time any barber can start the chosen service. Drives the "Next open chair" box in the header.
- `renderAll()`: re-renders the day chips, barber chips, times table, appointment card and header box. **All state changes go through it.**
- `state`: `{ service, dayKey, barber, start, only, booked }`. `only` is the barber filter.

The booking flow has three steps: service, then barber and time, then details. Details are first name, last name, mobile and email, all required, plus optional notes. Mobile must have 10 digits and email must look like `name@example.com`; both are checked when the booking button is pressed.

## Design system (keep to it)

- **Theme:** colour tokens on `:root`: mint tile `--tile`, card paper `--paper`, bottle-green ink `--ink`, pomade-red accent `--oxblood`, plus `--rule`, `--booked` and `--focus`. Dark mode redefines the same tokens under `prefers-color-scheme: dark` (guarded by `:root:not([data-theme="light"])`) and again under `:root[data-theme="dark"]`. Never use a literal colour in a component rule.
- **Type:** `--font-sign` (Big Shoulders Display) for headings and prices, `--font-body` (Public Sans) for everything else, and `--font-hand` (Caveat) **only** for handwritten values on the appointment card and the chosen time slot.
- **Look:** the times table is drawn like a paper appointment book, and the summary is drawn like a paper appointment card. Avoid generic SaaS cards, gradients, emoji and all-caps labels.
- **Copy:** plain, active, sentence case. Each button names what it does ("Book 2:30 pm with Dee"). Errors say how to fix the problem.
- **Quality floor:** works at 400px wide with no sideways scrolling. Visible focus outlines, `prefers-reduced-motion` respected, safe-area insets handled for phones.
- **Breakpoints:** 860px (the desk stacks and the phone bottom bar appears) and 520px (form fields go to one column and the times table compacts).

## Conventions

- Keep it a single static file unless the backend work requires otherwise. Don't add a framework or build step without asking.
- Put new data in the constants above, not hard-coded into the markup.
- Commit messages: a short summary line, a blank line, then a body explaining why.
- Example data is clearly fictional: the shop, barbers, phone number (555) and address. Don't present it as real.

## Known gaps

1. No persistence or backend. Booked times are generated.
2. The "confirmation email" and "text reminder" mentioned in the copy are **not sent**.
3. Prices are hard-coded, so a price change in Square won't reach the site until it's integrated (`docs/backend-plan.md`).
4. There's no barbers' view of the day's bookings.

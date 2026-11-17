# Backend plan

This records the decisions made so far about turning the static booking page into a real booking system, and why.

## Why a backend is needed at all

Whatever the backend ends up being, something on a server has to:

1. **Prevent double bookings.** Checking that a time is free and saving the booking has to happen as one step, in one place that is the source of truth.
2. **Keep secrets.** Square, HubSpot and email services all need private access keys. The site is public static files, so anything in the page can be read by anyone.
3. **Serve availability.** The times table needs to know which times are actually taken.
4. **Send messages.** Confirmation emails and reminder texts the evening before.
5. **Validate and protect.** Spam and fake bookings, checking details, and keeping consent records.

## Why a CRM isn't a substitute

HubSpot or Salesforce are good at storing contacts and running marketing (rebooking reminders, promotions, reviews). They aren't scheduling systems: they can't check whether a time is already taken and refuse a second booking for it. Writing to one from the browser would also expose the access key. A CRM can be **added** as a place that receives copies of client details, but it can't be the booking backend. Salesforce is far more than a three-chair shop needs; HubSpot's free CRM would be enough if one is wanted.

## Options considered

| Option | Verdict |
|---|---|
| A. Small custom backend (for example Supabase plus a server function) with a HubSpot copy | Full control over data and design. More to build and run. |
| **B. A ready-made booking service, with Square Appointments as the choice** | **Chosen.** Square handles bookings, client records, payments online and in the shop (card readers, tips), deposits and no-show fees, and reminders. It also has public developer tools, so this page can stay the front end. |
| C. CRM only | Rejected: it can't prevent double bookings and still needs a server to hold its key. |

Fresha and Booksy were also considered. They have built-in payments and directory apps that bring in new clients, but they mostly replace a custom page with their own booking widget. Cal.com takes payments through Stripe but only online, with no in-shop register.

## Target design with Square

```
Booking page ──fetch──► server functions (hold the Square key)
                            ├── GET  /api/barbers       Barber profiles: photo, name, experience, working hours
                            ├── GET  /api/services      Square's service list: names, lengths, one price each, and which barbers offer it
                            ├── GET  /api/availability  Square's free times for a service, day and barber
                            └── POST /api/bookings      Create the booking in Square (and the customer if they're new)
```

- **Server functions:** any serverless host works (for example Cloudflare Workers, Vercel or Netlify Functions). GitHub Pages can't run server code, so the functions live elsewhere and the page calls them.
- **Prices come from Square.** When a price changes in the Square Dashboard, the site picks it up on the next page load, or within a few minutes if the list is cached. Optionally, Square can notify the server when the service list changes, which Square calls a webhook.
- **The customer is charged Square's current price.** The page should re-check the price before confirming, and tell the customer if it changed since they opened the page.
- **Messages:** Square sends confirmations and reminders, which replaces the promises the page currently makes but doesn't keep.

### Account setup and pricing (decided 2026-11-16)

- **The barbers work for the shop.** There is **one Square account** for the shop, and each barber is a team member on it. Payments go to the shop, which pays the barbers.
- **One price per service, whatever the barber.** A skin fade costs the same with Dee, Marco or Sam. Prices aren't set per barber in Square, and the front end keeps showing one price per service.
- **Who changes prices:** the owner or a manager, in the Square Dashboard. Barbers don't need permission to edit services.
- **Assigning services:** each service is assigned to the barbers who offer it. That decides which columns can take a booking for it, not its price.

### Barber profiles come from the backend (decided 2026-11-16)

The page won't hard-code barbers any more. It loads them from `GET /api/barbers` on page load and builds the barber filter, the times-table columns and the Barbers section from that list. Adding, removing or editing a barber then needs no change to the site.

Each profile has:

| Field | Shown as | Where it comes from |
|---|---|---|
| `id` | Not shown; links the profile to bookings | The barber's Square team member ID |
| `name`, `shortName` | "Dee Okafor", and "Dee" in buttons and column headings | The display name on the barber's Square booking profile |
| `photo` | Round portrait (header of their column, filter button, Barbers section) | The photo on the barber's Square booking profile if one is set, otherwise an image the shop uploads. If there's none, the page shows initials. |
| `experience` | "12 years in the chair" | Store the **year they started** (for example `2014`), not a number of years, so it stays right every year. The server works out the years. Square has no field for this, so it's kept in the shop's own small settings store next to the Square team member ID. |
| `specialty`, `bio` | "Fades", and the short paragraph in the Barbers section | The description on the Square booking profile, or the shop's settings store |
| `availability` | Which days and hours the barber works, and whether they can be booked online at all | Their Square booking setup: working hours, days off, and whether they're bookable online |

Example response:

```json
[
  {
    "id": "TMa1b2c3",
    "name": "Dee Okafor",
    "shortName": "Dee",
    "photo": "https://…/dee.jpg",
    "startedYear": 2014,
    "experienceYears": 12,
    "specialty": "Fades",
    "bio": "Skin fades, tapers and sharp line-ups. Bring a photo of the cut you want.",
    "bookable": true,
    "workingHours": { "2": [[540, 1140]], "3": [[540, 1140]], "6": [[480, 960]] }
  }
]
```

`workingHours` uses the same format as the page's `HOURS` constant: weekday (`0` = Sunday) to a list of `[startMin, endMin]` blocks, so a lunch break is two blocks.

**Working hours and free times are separate.** A barber's working hours say when they're in the shop. Which of those times are still free comes from `GET /api/availability`, which asks Square for its open times. The page uses working hours to grey out days a barber is off ("Dee is off on Sundays"), and availability for the times table.

**Rules for the page:**
- Only show barbers marked `bookable`. Someone on holiday or not taking online bookings is hidden from the filter and the times table, but can stay in the Barbers section.
- If the profile list fails to load, show a clear message with a retry button. Don't fall back to made-up barbers.
- Keep a copy of the list for a few minutes; profiles change rarely.

**Check before building:** from general knowledge, Square's booking profiles hold a display name, description and photo, and Square can return a barber's free times. How each barber's regular working hours can be read through Square's developer tools needs confirming. If they can't be, keep them in the shop's settings store alongside the start year.

Prices per barber were considered and turned down. If the shop ever wants them, Square can set a price for each barber on the same account, and the front end would then need "from $X" prices.

### Check before building

Feature details and fees here come from general knowledge, not a check of Square's current documentation. Before building, check Square's current plans (which tier the shop needs for three team members), its fees, and what's available in the shop's country.

## Front-end changes to prepare

1. Replace the hard-coded `SERVICES`, `BARBERS` and `HOURS` with a single loader that reads from a data source. Use the current constants as a local stand-in until the API exists. `BARBERS` should take the `/api/barbers` shape above (`startedYear`, `workingHours`, `bookable`).
2. Build the barber filter, times-table columns and Barbers section from the loaded list, and grey out times outside each barber's working hours.
3. Replace the fake `isTaken` hash with data from `/api/availability`.
4. Make the booking button send to `/api/bookings`. Handle "that time was just taken" by refreshing the times table and showing a clear message.
5. Add loading and error states to the times table.

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
                            ├── GET  /api/services      Square's service list: names, lengths, prices (per barber if used)
                            ├── GET  /api/availability  Square's free times for a service, day and barber
                            └── POST /api/bookings      Create the booking in Square (and the customer if they're new)
```

- **Server functions:** any serverless host works (for example Cloudflare Workers, Vercel or Netlify Functions). GitHub Pages can't run server code, so the functions live elsewhere and the page calls them.
- **Prices come from Square.** When a price changes in the Square Dashboard, the site picks it up on the next page load, or within a few minutes if the list is cached. Optionally, Square can notify the server when the service list changes, which Square calls a webhook.
- **The customer is charged Square's current price.** The page should re-check the price before confirming, and tell the customer if it changed since they opened the page.
- **Messages:** Square sends confirmations and reminders, which replaces the promises the page currently makes but doesn't keep.

### Prices per barber

Which setup to use depends on how the shop is run:

- **Barbers who work for the shop:** one Square account, with a default price per service and a different price and length for each barber where needed. The owner or a manager edits prices.
- **Barbers who rent their chairs:** each barber has their own Square account with their own prices and payouts. A combined page then has to read from several accounts, with each barber's permission.

**Still to decide:** which of these the shop is.

If prices per barber are used, the front end should show "from $X" in the service list, each barber's price at the top of their column, and the chosen barber's price on the appointment card.

### Check before building

Feature details and fees here come from general knowledge, not a check of Square's current documentation. Before building, check Square's current plans (which tier includes prices per barber), its fees, and what's available in the shop's country.

## Front-end changes to prepare

1. Replace the hard-coded `SERVICES`, `BARBERS` and `HOURS` with a single loader that reads from a data source. Use the current constants as a local stand-in until the API exists.
2. Replace the fake `isTaken` hash with data from `/api/availability`.
3. Make the booking button send to `/api/bookings`. Handle "that time was just taken" by refreshing the times table and showing a clear message.
4. Add loading and error states to the times table.

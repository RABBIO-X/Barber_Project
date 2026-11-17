# Barber_Project

A booking page for Kettle Street Barbers, a three-chair barbershop. It's one static HTML file with no build step and no dependencies apart from Google Fonts.

**Live site:** https://rabbio-x.github.io/Barber_Project/

## What it does

- **Next open chair:** the header shows the earliest free time for the chosen service. "Take this time" fills in the booking for you.
- **Three booking steps:** pick a service, pick a barber and a time, then enter your details (first and last name, mobile, email, and optional notes for the barber).
- **Up to two weeks ahead:** a scrolling day picker with arrow buttons shows today plus the next 14 days.
- **Times table:** one column per barber, like a paper appointment book. A filter shows all barbers or just one.
- **Appointment card:** fills in as you choose and gets a "Booked" stamp when you confirm.
- **Barbers:** profiles with photos. "See Dee's times" filters the times table to that barber.
- **Hours and address:** opening hours (today in bold), directions, and the shop phone number with a copy button.
- **Phones and themes:** works on phones, with a bottom bar that keeps the next step within reach. Light and dark themes follow the system setting, and it works with a keyboard and screen readers.

## Run it

Open `index.html` in a browser. To serve it locally instead:

```sh
python -m http.server 8000
# then visit http://localhost:8000
```

## Customize

All the content is near the top of the `<script>` block in `index.html`:

- `SERVICES`: name, description, length in minutes, price
- `BARBERS`: name, specialty, years, bio, and `photo` (a path in `images/`). Without a photo, the barber's initials show.
- `HOURS`: opening hours per weekday, in minutes from midnight (`null` means closed)
- `BOOK_AHEAD_DAYS`: how many days ahead people can book (default 14)

The barber photos in `images/` were generated with AI in Canva. Replace them with photos of the real barbers, keeping the same file names.

## Current limits

- **No backend.** Bookings live only in the visitor's browser and disappear when the page reloads, so barbers can't see them yet.
- **Example data.** Booked times are generated, so the book looks busy. The shop, barbers, address and phone number are fictional.
- **No real messages.** The page mentions a confirmation email and a reminder text, but neither is sent.

The planned backend is Square Appointments. See [docs/backend-plan.md](docs/backend-plan.md) for why, and for how prices, availability and bookings will connect.

## Docs

- [CLAUDE.md](CLAUDE.md): code map, design system and conventions for contributors (and for Claude Code)
- [docs/backend-plan.md](docs/backend-plan.md): why the page needs a backend, the options considered, and the Square integration design

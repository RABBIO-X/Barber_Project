# Barber_Project

A booking page for Kettle Street Barbers, a three-chair barbershop. It's one static HTML file with no build step and no dependencies apart from Google Fonts.

## What it does

- **Next open chair:** the header shows the earliest free time for the chosen service. "Take this time" fills in the booking for you.
- **Three booking steps:** pick a service, pick a barber and a time from each barber's book for the day, then enter your name and mobile number.
- **Appointment card:** fills in as you choose and gets a "Booked" stamp when you confirm.
- **Barbers:** profiles with picture placeholders. "See Dee's times" filters the book to that barber.
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
- `BARBERS`: name, specialty, years, bio, and an optional `photo` (an image URL or data URI). Without a photo, the barber's initials show.
- `HOURS`: opening hours per weekday, in minutes from midnight (`null` means closed)

## Current limits

- **No backend.** Bookings live only in the visitor's browser and disappear when the page reloads, so barbers can't see them yet.
- **Example data.** Booked times are generated, so the book looks busy.
- **No real messages.** No text reminder is sent.

Next step: add a small API and database (for example Supabase, Firebase, or a Node/Python server) to store bookings, plus a barbers' view to manage the day's appointments.

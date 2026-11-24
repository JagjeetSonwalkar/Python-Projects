# Problem Statement & Project Proposal

## 1. Background

SwiftTravel is a (fictional) transport company that operates buses between
cities — for example Pune → Mumbai and Pune → Nashik. Like many small
transport operators, it currently manages passenger reservations manually
(pen-and-paper registers, phone calls, or a spreadsheet maintained by hand).

This project takes that real-world scenario and asks: *what would it take to
replace the manual process with a small, correct, well-structured Python
program?*

---

## 2. The Problem

Manual reservation management breaks down in predictable ways as soon as more
than a handful of bookings are involved:

| Manual process pain point | Concrete failure mode |
|---|---|
| No single source of truth for seat status | The same seat gets promised to two different passengers (**double-booking**) |
| Seat availability tracked by memory or a paper chart | Staff cannot say, at a glance, how many seats are actually free for a given date |
| Fares calculated by hand | Age-based discount rules are applied inconsistently or forgotten |
| Booking records kept on paper or scattered notes | Records get lost, damaged, or are simply hard to search through |
| No structured way to validate what a customer types/says | A mistyped age, an invalid seat number, or a nonsense date crashes the process (or worse, gets accepted silently) |
| No waiting list mechanism | When a passenger cancels, the freed seat isn't offered to anyone — it just sits empty, or is handed out on a first-come-whoever's-in-line-that-day basis rather than fairly |
| Nothing persists between sessions | If today's register is lost, so is today's business |

None of these are exotic problems — they're the everyday failure modes of
**any manual booking system**, whether it's a bus company, a small clinic
scheduling appointments, or a restaurant taking table reservations. That's
what makes this a good practice problem: solving it well requires real
data-structure and software-design thinking, not just "print some text."

---

## 3. What We're Actually Solving

Restated as concrete, testable problems the program must handle correctly:

1. **Prevent double-booking** — a seat that is `CONFIRMED` for a given bus
   *and* a given travel date must never be offered to a second passenger.
2. **Make availability checkable at any time** — for any bus and any date
   within the booking window, the system must be able to say exactly which
   seats are free and which are taken.
3. **Calculate fares consistently** — age-based discount rules must be
   applied the same way, every time, without relying on a human doing
   mental math.
4. **Never lose a booking** — once confirmed, a booking must be
   retrievable by ID, searchable by passenger name, and must survive the
   program being closed and reopened.
5. **Reject bad input without crashing** — non-numeric ages, invalid
   journey IDs, out-of-range seats, and out-of-window dates must be caught
   and reported back to the user, not allowed to corrupt data or crash the
   program.
6. **Handle cancellations fairly** — a cancelled seat should become
   available again immediately, and if people are waiting for that
   journey/date, the *first* person to have joined the waiting list should
   get it automatically (FIFO), without disturbing the order of anyone
   still waiting.
7. **Restrict how far in advance people can book** — the business only
   wants to accept bookings for the near future (today through the next
   7 days), not arbitrary future dates.

---

## 4. Proposed Solution

Build a single-file, console-based Python application (`bus_reservation.py`)
that models the problem directly:

- **A custom doubly linked list** (`Node` / `DoublyLinkedList`) is the core
  data structure for both confirmed bookings and the waiting list. It's the
  right structural fit because the two operations that matter most —
  *append to the back* (a new booking / a new waiting-list entry) and
  *remove from the front in order* (serving the waiting list FIFO) — are
  exactly what a doubly linked list does well, and cancelling an arbitrary
  booking is an O(1) unlink once the node is found.
- **Seat availability is computed per `(bus_id, journey_date)` pair**, using
  set arithmetic (`all seats - booked seats`), so the same bus can be full
  on one date and empty on another — mirroring how real transport
  scheduling works.
- **CSV files** (`bookings.csv`, `waitlist.csv`) give the system durable,
  human-readable storage using only the standard library, satisfying the
  "must not lose data between sessions" requirement without adding a
  database dependency.
- **`Passenger` and `Booking` classes** give the data proper structure
  (rather than passing raw tuples/dicts around) and centralize how a
  booking is displayed, serialized to a CSV row, and rebuilt from one.
- **Input is validated at the point of entry** with `try/except` and
  bounded prompts, so bad input is caught immediately instead of
  propagating into stored data.
- **A rolling 8-day booking window** (`BOOKING_WINDOW_DAYS`) is enforced
  centrally, so every entry point (booking, viewing seats) respects the
  same early-booking policy.

---

## 5. Scope

**In scope**
- Console/text interface only (no GUI, no web front end)
- Two pre-configured bus routes (easily extendable via the `BUSES` dict)
- Single transport company, single currency (₹)
- Local file-based storage (CSV)
- Single-user, single-process usage (no concurrent multi-user access)

**Out of scope** (see [README → Possible extensions](../README.md#-possible-extensions))
- Payment processing
- Multi-leg / connecting journeys
- Seat classes or differentiated pricing tiers
- Admin interface for adding/editing buses at runtime
- Authentication / multi-user accounts
- A database backend or network access

---

## 6. Who This Is For

- **Passengers** — book, cancel, search, and view their own bookings
  through a simple menu.
- **Transport-company staff** — could use the same tool at a counter to
  check seat availability and process bookings/cancellations for walk-in
  customers.
- **The developer (this project's actual audience)** — a practice exercise
  in taking a fuzzy real-world problem and turning it into a well-structured
  Python program using linked lists, OOP, file I/O, and exception handling.

---

## 7. Success Criteria

The project is considered successful if:

- [x] No two confirmed bookings can ever occupy the same seat on the same
      bus/date.
- [x] Fares are calculated correctly for all three age brackets
      (`<12`, `12–59`, `60+`).
- [x] Invalid input (bad age, bad seat, bad date, bad journey ID) never
      crashes the program.
- [x] Cancelling a booking releases the seat and, if applicable,
      automatically assigns it to the next waiting-list passenger in FIFO
      order.
- [x] All bookings and waiting-list entries persist across program
      restarts via CSV.
- [x] Bookings can be found by exact ID or partial passenger-name search.
- [x] Booking dates are restricted to the configured rolling window.

Each of these is verified with real captured output in
[docs/RESULTS.md](RESULTS.md).

---

## 8. Constraints & Assumptions

- Python 3.7+ with only the standard library — no external dependencies.
- Single machine, single user at a time; no file-locking or concurrency
  control is implemented.
- Dates/times are treated as naive (no timezone handling) since the whole
  system models one company operating in one region.
- Bus routes, fares, and capacities are hardcoded in `BUSES` for this
  exercise, but the data model doesn't assume there are only two — new
  routes can be added by extending the dictionary with no other code
  changes.

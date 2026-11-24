# 🚌 SwiftTravel — Console-Based Bus Reservation System

A console-based Python application that simulates a real-world bus/railway
reservation service: passengers can view journeys, check seat availability
by date, book tickets, cancel bookings, search records, and get auto-enrolled
from a waiting list the moment a seat frees up.

Built as a Python practice project focused on **problem-solving, clean code
structure, and core data structures** — not a GUI or web app.

📄 **New here?** Start with [docs/PROBLEM_STATEMENT.md](docs/PROBLEM_STATEMENT.md)
for the "what problem are we solving and why" writeup before diving into the code.

![Main menu](docs/screenshots/01_main_menu.png)

---

## ✨ Features

- **View buses** — all active routes, timings, fares, and total capacity.
- **Date-aware seat map** — every bus/date combination has its own seat map,
  so B101 can be full on Monday and wide open on Tuesday.
- **Early-booking window** — tickets can only be booked for **today through
  the next 7 days** (8-day rolling window); anything outside that range is
  rejected with a clear message.
- **Book a ticket** — full validation on name, age, journey ID, date and
  seat before confirming.
- **Age-based fare rules**:
  | Age range | Rule |
  |---|---|
  | Under 12 | 50% discount |
  | 12–59 | Full fare |
  | 60+ | 30% discount |
- **Unique booking IDs** — auto-generated (`ST1001`, `ST1002`, …).
- **Cancel a ticket** — releases the seat immediately.
- **Automatic waiting list** — if a bus/date is full, a passenger can join
  a FIFO waiting list. The moment a seat is cancelled, the **first**
  waiting passenger is automatically booked into it — without disturbing
  anyone else's position in the queue (see the ["Final Challenge"](docs/RESULTS.md#5-cancellation--automatic-waiting-list-allocation-final-challenge) scenario).
- **Search** — partial, case-insensitive name search or exact booking-ID
  lookup.
- **Persistent storage** — all bookings and the waiting list are saved to
  CSV files and reloaded automatically on the next run.
- **Robust input handling** — invalid ages, seat numbers, journey IDs, and
  dates are caught and re-prompted instead of crashing the program.

---

## 🧠 Data structures & design

| Requirement | Implementation |
|---|---|
| Core booking store | **Custom doubly linked list** (`Node` / `DoublyLinkedList`) — not a Python `list`. Chosen so that FIFO waiting-list processing (append to the back, pop from the front) and mid-list cancellation (unlink a node in O(1) once found) map directly onto the underlying structure. |
| Waiting list | Same `DoublyLinkedList` class, reused — passengers are appended when they join and removed from the front as seats free up, which is exactly what a doubly linked list is good at. |
| Persistent storage | **CSV files** (`bookings.csv`, `waitlist.csv`) via Python's built-in `csv` module, read/written with `with open(...)`. |
| Journeys | Dictionary keyed by bus ID (`BUSES`), each value itself a dictionary of route/time/fare/capacity. |
| Passenger / Booking | Two OOP classes with `__init__`, instance attributes, and methods (`display()`, `to_row()`, `from_row()`). |
| Seat availability | Computed per `(bus_id, journey_date)` using set operations (`set(range(1, total+1)) - booked`). |
| Validation & errors | `try/except` around all numeric/user input; graceful re-prompting instead of crashing. |

---

## 📸 Screenshots

| View Buses | View Seats (date-aware) |
|---|---|
| ![View buses](docs/screenshots/02_view_buses.png) | ![View seats](docs/screenshots/03_view_seats.png) |

| Book Ticket (fare rule applied) | Cancel → Waiting-list auto-fill |
|---|---|
| ![Book ticket](docs/screenshots/04_book_ticket.png) | ![Cancel and autofill](docs/screenshots/05_cancel_and_waitlist_autofill.png) |

| Search |
|---|
| ![Search](docs/screenshots/06_search.png) |

More detail, plus the raw output and resulting CSV data for every scenario
above, is in **[docs/RESULTS.md](docs/RESULTS.md)**.

---

## 🗂 Project structure

```
bus-reservation-system/
├── bus_reservation.py      # entire application (single file)
├── requirements.txt        # no external deps — stdlib only
├── LICENSE
├── .gitignore
├── README.md
└── docs/
    ├── PROBLEM_STATEMENT.md # what problem this solves, scope, success criteria
    ├── RESULTS.md           # test scenarios with real captured output
    ├── sample_bookings.csv  # example of the on-disk data format
    └── screenshots/

```

---

## 🚀 Getting started

### Requirements
- Python 3.7 or later
- No external packages — only the standard library (`csv`, `os`, `datetime`)

### Run it

```bash
git clone https://github.com/<JagjeetSonwalkar>/bus-reservation-system.git
cd bus-reservation-system
python3 bus_reservation.py
```

On first run, `bookings.csv` and `waitlist.csv` will be created automatically
in the working directory as soon as you make a booking.

### Menu

```
=====================================
 SWIFTTRAVEL - BUS RESERVATION SYSTEM
=====================================
1. View Buses
2. View Available Seats
3. Book Ticket
4. Cancel Ticket
5. View Booking
6. Search Passenger / Booking
7. Exit
```

---

## 🧪 Sample data

Two buses are pre-configured in `BUSES` at the top of `bus_reservation.py`:

| Bus ID | Route | Departure | Arrival | Seats | Fare |
|---|---|---|---|---|---|
| B101 | Pune → Mumbai | 08:00 AM | 12:00 PM | 10 | ₹450 |
| B102 | Pune → Nashik | 09:30 AM | 01:00 PM | 12 | ₹350 |

Add more by extending the `BUSES` dictionary — no other code changes needed.

---

## 🛣 Possible extensions

- Multiple pickup points / intermediate stops per route
- Seat classes (AC / non-AC, window / aisle) with differentiated pricing
- Admin mode to add/remove buses at runtime
- Email/SMS-style notification stub when a waiting-list booking is confirmed
- Unit tests (`pytest`) for the linked-list and fare-calculation logic
- Migrate storage to SQLite while keeping the same linked-list in-memory model

---

## 📄 License

Released under the [MIT License](LICENSE).

## 🙋 About

This project was built as a Python fundamentals practice exercise:
translating a real-world reservation problem into clean, well-structured
Python using linked lists, file I/O, exception handling, and OOP.

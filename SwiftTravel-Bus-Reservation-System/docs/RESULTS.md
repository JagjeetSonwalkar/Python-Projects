# Test Results & Sample Runs

This document records the manual test scenarios used to verify the application,
along with real captured output. All transcripts below were produced by
actually running `bus_reservation.py` — nothing here is hand-written.

---

## 1. Fare calculation by age

| Passenger    | Age | Base Fare (B101) | Rule Applied              | Final Fare |
|--------------|-----|-------------------|----------------------------|------------|
| Rahul Sharma | 24  | ₹450              | Age 12–59 → full fare      | ₹450.00    |
| Sunita Verma | 65  | ₹450              | Age 60+ → 30% discount     | ₹315.00    |
| (child, e.g.)| 8   | ₹450              | Age < 12 → 50% discount    | ₹225.00    |

```
Booking ID : ST1002
Passenger  : Sunita Verma
Journey    : B101
Route      : Pune -> Mumbai
Date       : 2026-09-06
Seat       : 05
Fare       : Rs.315.00
Status     : CONFIRMED
```

---

## 2. Seat-conflict prevention

Attempting to book a seat that is already `CONFIRMED` on the **same bus and
same date** is rejected before any file write happens:

```
Available seats on 2026-09-06: 02, 03, 04, 05, 06, 07, 08, 09, 10
Select seat: 1
That seat is already booked or invalid. Please try again.
```

The same seat number **is** bookable again on a *different* date, since
availability is tracked per `(bus_id, journey_date)` pair, not per bus alone.

---

## 3. Input validation / exception handling

```
Enter age: abc
Invalid input. Age/seat must be a number. Please try again.
Enter age: 45
```

Non-numeric ages, out-of-range ages, invalid journey IDs, invalid seat
numbers, and out-of-window dates are all caught and re-prompted without
crashing the program.

---

## 4. Early-booking window enforcement

Dates outside the rolling 8-day window are rejected:

```
Enter date (YYYY-MM-DD) or option number [1-8], 'c' to cancel: 2026-12-25
Bookings are only open for the next 8 days (2026-09-06 to 2026-09-13).
Please choose a date in that range.
```

---

## 5. Cancellation + automatic waiting-list allocation (Final Challenge)

**Scenario:** Bus B101 has 10 seats, all 10 are booked for 2026-09-06.
Two more passengers (Deepak Rao, Meena Iyer) try to book and are added to
the waiting list instead. Two existing passengers then cancel.

**Expected behaviour (per the project's "Final Challenge" spec):**
1. Freed seats are detected.
2. The **first** passengers in the waiting list (FIFO) are pulled off.
3. They are assigned the freed seats and new bookings are created.
4. They are removed from the waiting list.
5. Anyone still waiting keeps their original relative order.

**Actual captured output:**

```
--- Cancel Ticket ---
Enter Booking ID (or 'c' to cancel): ST1003
Booking found.
Passenger: Passenger3
Journey: B101 on 2026-09-06
Seat: 03
Cancel booking? (yes/no): yes
Booking cancelled successfully.
Seat 03 is now available.
Waiting list: seat 03 on B101 (2026-09-06) auto-assigned to Deepak Rao (Booking ID: ST1011).

--- Cancel Ticket ---
Enter Booking ID (or 'c' to cancel): ST1005
Booking found.
Passenger: Passenger5
Journey: B101 on 2026-09-06
Seat: 05
Cancel booking? (yes/no): yes
Booking cancelled successfully.
Seat 05 is now available.
Waiting list: seat 05 on B101 (2026-09-06) auto-assigned to Meena Iyer (Booking ID: ST1012).
```

**Resulting `bookings.csv`** (see [`sample_bookings.csv`](./sample_bookings.csv) for the full file):

| booking_id | name        | seat | status    |
|------------|-------------|------|-----------|
| ST1003     | Passenger3  | 03   | CANCELLED |
| ST1005     | Passenger5  | 05   | CANCELLED |
| ST1011     | Deepak Rao  | 03   | CONFIRMED |
| ST1012     | Meena Iyer  | 05   | CONFIRMED |

The waiting list was drained in FIFO order and the file persisted correctly
— confirmed by restarting the program and re-reading the same data.

---

## 6. Search

```
--- Search Passenger / Booking ---
Enter passenger name or Booking ID: Rahul

Search Results (1 found)
---------------------------------------------
Booking ID : ST1001
Passenger  : Rahul Sharma
Journey    : B101  Date: 2026-09-06
Seat       : 04
Status     : CANCELLED
---------------------------------------------
```

Partial, case-insensitive name matching works, as does exact booking-ID
lookup, per the requirement in the project brief.

---

## 7. Persistence across restarts

`bookings.csv` and `waitlist.csv` are rewritten after every state-changing
action (`book`, `cancel`, waiting-list auto-fill). Killing and restarting
the program reloads all bookings and any still-waiting passengers exactly
as they were left — verified by inspecting the CSV files directly (see
`sample_bookings.csv`).

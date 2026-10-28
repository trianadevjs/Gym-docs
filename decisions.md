# Decisions

Log of product and technical decisions and the reasoning behind them.

## 2026-10-27 — Initial decisions

### 1. MVP centered on Reception
Reception is the operational core and generates the data (members, memberships,
payments) that the manager and owner reports need. Reports and member features come
in later phases. See [MVP scope](mvp-scope.md).

### 2. Multi-gym model from day one
Operational data is tied to a `Gym` from the start, even with a single location.
Adding a `gymId` later would require migrating every existing record.

### 3. Memberships are per gym
A membership is valid at one location only. This is the simplest option and matches
the per-gym revenue and active-membership reports. Franchise-wide plans can be
added later if needed.

### 4. PostgreSQL + Prisma
The data is highly relational (gym → members → memberships → payments → attendance),
and period reports are a natural fit for SQL. Prisma gives a typed client and simple
migrations.

### 5. Own JWT auth with four fixed roles
`OWNER`, `MANAGER`, `RECEPTIONIST`, `MEMBER`, enforced with Nest guards. With a fixed
set of roles an external auth provider is not needed, which avoids extra cost and
vendor dependency.

### 6. Attendance recorded on entry only, exit estimated
Members won't reliably scan on the way out, and there is no turnstile hardware.
Exit is estimated after a fixed window (e.g. 2 hours) to compute current occupancy.

### 7. Configurable plan catalog per gym
Prices change and can differ by location. A `Plan` table per gym avoids code changes
whenever a price changes.

### 8. A single PWA for all roles
`gym-manager` is one web app for owner, manager, reception and members, installable as
a Progressive Web App so it can be used partly offline.

### 9. Offline is read-only, plus the member's QR code
Recording payments offline risks sync conflicts that would break the cash cut. Offline
covers showing the QR code, viewing attendance, and searching members from cache. All
writes require a connection.

### 10. Frontend stack: React + Vite + `vite-plugin-pwa`
A SPA is the most direct base for a PWA with a service worker and offline cache.
Next.js adds server-side rendering, which brings little here and makes offline harder.

### 11. Documentation in English
All documentation in this repository is written in English.

### 12. Multi-tenant SaaS
The product is sold to many gyms and franchises, not built for a single one. Each client
is an `Organization` (tenant) with one or more gyms. A single database and deployment
serve every tenant, and every tenant-scoped table carries an `organizationId` that the
API filters by. This extends decision 2: the multi-gym model now sits under an
organization. See [Domain model](domain-model.md).

### 13. Product brand, no white-label
Every client sees the product's brand; a gym appears only as its name and data. This
keeps one design for all clients. "Gym Manager" is a working name; there is no final
name, logo or color palette yet.

### 14. Spanish (Mexico) interface, MXN
The market is Mexico. The interface is in Spanish and prices are in MXN, stored as
integer cents. Code and documentation stay in English.

### 15. Reception works on PC or tablet
The front desk device varies by gym, so reception screens must work with mouse,
keyboard and touch.

### 16. Membership periods are inclusive calendar days
A membership's `startsOn` and `endsOn` are dates in the gym's time zone, and `endsOn` is
the last valid day. A plan period ends at `start + duration − 1 day` (a 1-month plan
starting 28 Oct ends 27 Nov; a 1-day visit starts and ends the same day). A payment
made while the membership is still valid (`endsOn` is today or later) starts the new
period the day after `endsOn`, so no paid day is lost; a payment for an expired
membership or a new member starts today. The API and the app must use this same rule
so the next payment date shown at reception matches what is stored.

### 17. Cash cut: blind count since the last cut, with a fixed float
A cash cut covers every payment recorded at the gym since the previous cut, whoever
took it, so it works for one or several shifts a day without configuring schedules.
Each gym has a fixed cash float that stays in the drawer. Staff counts all the cash in
the drawer and enters it as one total **before** seeing what is expected (blind count),
so the count is not adjusted to match; once confirmed, the count cannot be edited.
Expected cash is the float plus the period's cash payments, and staff withdraws the
counted amount minus the float. Card payments are listed and totalled but not counted.
A difference does not block the cut, but closing it requires a note explaining it.

### 18. Staff sign-in on shared front-desk devices
Staff sign in with email and password. Because an email is unique only within an
organization, the API checks the password first and, when the same credentials match
accounts in more than one organization, asks which one to enter; it never reveals which
organizations an email belongs to before the password is verified. Managers and
receptionists land on their own gym; owners, who have no gym, pick one each time they
sign in. Member accounts are told that member features come later.

Front-desk devices are shared by the whole team, so each device remembers up to five
people who signed in on it (name, role, gym and email, never a password) and the next
sign-in is choosing a name plus the password. Anyone can remove a person from that list.
"Salir" ends the session; there is no automatic sign-out in the MVP.

The access token lives in memory and the refresh token in an httpOnly cookie, so no
token is stored where page scripts can read it. The signed-in profile (who and which
gym) is cached so a device reopened offline stays usable read-only; signing in itself
requires a connection. Forgotten passwords are reset by a manager or owner; there is no
self-service reset or PIN in the MVP.

### 19. Who manages gyms, plans and staff
Owners manage every gym of their organization: they create gyms and set each one's
name, address and cash float, and they manage the plans and staff (managers and
receptionists) of any gym. Managers manage the plans and receptionists of their own
gym and can read its details, but cannot change the float or manage other managers.
Receptionists have no access to the catalog. The API enforces these rules; the app only
hides what a role cannot do.

Plans are never deleted, because past payments reference them: they are retired and can
be reactivated, and reception only offers active plans. Two active plans of a gym cannot
share a name. A price change applies to charges from that moment on and never changes
past payments. A float change applies from the next cash cut; the cut in progress keeps
the float it opened with.

New staff accounts and password resets get a system-generated temporary password of 8
digits, shown once to the manager or owner to hand over in person. It is valid for 24
hours and works only to set the person's own password (at least 8 characters) at their
first sign-in. Retired staff cannot sign in, and their payments and cash cuts are kept.

## Open decisions

- **QR reading at the entrance:** the reception device's camera in the app, a USB
  scanner, or something else.
- **Positioning:** what sets the product apart from other gym management software.

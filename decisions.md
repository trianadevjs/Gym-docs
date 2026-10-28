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

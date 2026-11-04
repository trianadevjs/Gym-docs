# Architecture

## Overview

```
gym-manager (PWA, all roles)  ──HTTP/JSON──▶  api-gym (NestJS)  ──▶  PostgreSQL
```

## Backend — `api-gym`

- **Framework:** NestJS 11 (TypeScript)
- **Database:** PostgreSQL
- **ORM:** Prisma (schema, typed client and migrations)
- **Auth:** our own JWT authentication (`@nestjs/jwt`) with access and refresh tokens
- **Authorization:** one role per user, enforced with Nest guards

### Multi-tenancy

The product is a SaaS: each client is an **organization** (a single gym or a franchise)
with one or more gyms. One database and one deployment serve every organization.

- Every tenant-scoped table carries an `organizationId`.
- The API resolves the organization from the authenticated user's token and filters
  every query by it. A user can never read or write another organization's data.
- Platform-level administration (creating organizations) is out of the MVP; the first
  organizations are created with a seed script.

### Roles

| Role           | Description                                           |
| -------------- | ----------------------------------------------------- |
| `OWNER`        | Owner of an organization, sees every gym it has       |
| `MANAGER`      | Manages a single gym                                  |
| `RECEPTIONIST` | Front desk staff of a single gym                      |
| `MEMBER`       | Gym customer                                          |

## Localization

- Interface language: Spanish (Mexico). Currency: MXN.
- Amounts are stored as integers in cents to avoid floating point errors.
- Dates are stored in UTC and shown in the gym's time zone.

## Frontend — `gym-manager`

- **One web app for all four roles.** Views and navigation depend on the user's role.
- **Product brand only.** Every client sees the product's brand; a gym appears only as
  its name and data (no white-label).
- **Devices:** reception uses a desktop PC or a tablet depending on the gym, so reception
  screens must work with mouse, keyboard and touch. Members use their phones.
- **Stack:** React + Vite (SPA) with `vite-plugin-pwa`
- **Progressive Web App:** installable on phones and desktops, with limited offline use.

### Offline behavior

Offline mode is **read-only, plus the member's QR code**:

| Works offline                                          | Requires a connection            |
| ------------------------------------------------------ | -------------------------------- |
| Member: show QR code, view attendance calendar/streak  | Recording payments               |
| Reception: search members and see next payment date (from cache) | Cash cut               |
|                                                        | Registering members, any write   |

Writes always need a connection so cash payments and cash cuts can never go out of sync.
An offline write queue can be added later if it turns out to be needed.

## Attendance and occupancy

Gym access is recorded **on entry only** (QR scan). An exit is **estimated** after a
fixed time window (e.g. 2 hours). "Current occupancy" counts entries whose estimated
exit has not passed yet. No exit scan or turnstile hardware is required.

Open decision: how the member's QR code is read at the entrance (the reception
device's camera in the app, a USB scanner, or something else).

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

### Roles

| Role           | Description                                      |
| -------------- | ------------------------------------------------ |
| `OWNER`        | Franchise owner, sees every gym                  |
| `MANAGER`      | Manages a single gym                             |
| `RECEPTIONIST` | Front desk staff of a single gym                 |
| `MEMBER`       | Gym customer                                     |

## Frontend — `gym-manager`

- **One web app for all four roles.** Views and navigation depend on the user's role.
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

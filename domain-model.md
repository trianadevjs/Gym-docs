# Domain Model

Initial model. Field lists are indicative and will be refined in the Prisma schema.

```
Organization 1──* Gym
Organization 1──* User
Gym 1──* User (staff: MANAGER, RECEPTIONIST)
Gym 1──* Plan
Gym 1──* Membership *──1 User (MEMBER)
Membership 1──* Payment
Gym 1──* Attendance *──1 User (MEMBER)
Gym 1──* CashCut 1──* Payment
```

## Tenancy

`Organization` is the tenant: a client of the SaaS (a single gym or a franchise). Every
other entity belongs to exactly one organization, directly or through its gym. Tables
store `organizationId` so the API can filter every query by tenant.

A user belongs to one organization. The same person signing up at two different client
organizations has two separate accounts.

## Entities

### Organization
A client of the SaaS.
- `id`, `name`, `createdAt`

### Gym
A location of an organization. Every operational record belongs to a gym from day one,
even while an organization has only one gym.
- `id`, `organizationId`, `name`, `address`, `timeZone`, `cashFloatCents` (the fixed float
  that stays in the reception drawer)

### User
Anyone who logs in. A single role per user.
- `id`, `organizationId`, `firstName`, `lastName`, `email`, `passwordHash`, `role`
- `gymId` — required for `MANAGER` and `RECEPTIONIST`; not used for `OWNER`
- `email` is unique within an organization

### Plan
A gym's configurable catalog of plans, so prices can change without code changes.
- `id`, `organizationId`, `gymId`, `name`, `durationUnit` (`DAY` | `WEEK` | `MONTH`),
  `durationCount`, `priceCents`, `active`

### Membership
A member's right to use **one specific gym** (memberships are per location).
- `id`, `organizationId`, `userId`, `gymId`, `startsOn`, `endsOn`
- `startsOn` and `endsOn` are calendar dates in the gym's time zone, both inclusive:
  `endsOn` is the last day the member may enter, and the next payment is due the day
  after. See decision 16 for how a payment extends them.

### Payment
A payment for a plan, which extends a membership. Amounts in MXN cents.
- `id`, `organizationId`, `membershipId`, `planId`, `amountCents`, `method` (cash, card,
  online), `receivedById` (staff user), `cashCutId`, `paidAt`

### Attendance
A gym entry recorded from a QR scan.
- `id`, `organizationId`, `userId`, `gymId`, `enteredAt`, `estimatedExitAt`

### CashCut
Closing of the cash register for a period or shift.
- `id`, `organizationId`, `gymId`, `number` (sequential per gym), `performedById`,
  `periodStart` (the previous cut's `periodEnd`), `periodEnd` (when the count was
  confirmed), `floatCents`, `cashSalesCents`, `cardSalesCents`, `expectedTotalCents`
  (float + cash sales), `countedTotalCents`, `note` (required when counted ≠ expected),
  `closedAt`
- Payments link to the cut whose period contains their `paidAt`. See decision 17.

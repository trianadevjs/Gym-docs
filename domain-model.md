# Domain Model

Initial model. Field lists are indicative and will be refined in the Prisma schema.

```
Gym 1──* User (staff: MANAGER, RECEPTIONIST)
Gym 1──* Plan
Gym 1──* Membership *──1 User (MEMBER)
Membership 1──* Payment
Gym 1──* Attendance *──1 User (MEMBER)
Gym 1──* CashCut 1──* Payment
```

## Entities

### Gym
A franchise location. Every operational record belongs to a gym from day one, even
while only one gym exists.
- `id`, `name`, `address`

### User
Anyone who logs in. A single role per user.
- `id`, `firstName`, `lastName`, `email`, `passwordHash`, `role`
- `gymId` — required for `MANAGER` and `RECEPTIONIST`; not used for `OWNER`

### Plan
A gym's configurable catalog of plans, so prices can change without code changes.
- `id`, `gymId`, `name`, `durationUnit` (`DAY` | `WEEK` | `MONTH`), `durationCount`,
  `price`, `active`

### Membership
A member's right to use **one specific gym** (memberships are per location).
- `id`, `userId`, `gymId`, `startsAt`, `endsAt` (the next payment date)

### Payment
A payment for a plan, which extends a membership.
- `id`, `membershipId`, `planId`, `amount`, `method` (cash, card, online),
  `receivedById` (staff user), `cashCutId`, `paidAt`

### Attendance
A gym entry recorded from a QR scan.
- `id`, `userId`, `gymId`, `enteredAt`, `estimatedExitAt`

### CashCut
Closing of the cash register for a period or shift.
- `id`, `gymId`, `performedById`, `periodStart`, `periodEnd`, `expectedTotal`,
  `countedTotal`, `createdAt`

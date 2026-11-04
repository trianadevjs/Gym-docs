# Gym Docs

Product and technical documentation for the gym franchise management system.

## Product overview

A multi-tenant SaaS for gyms and gym franchises in Mexico. Each client organization
can have one or more gyms (locations). It covers member registration, membership
payments, gym access via QR code, attendance tracking and reports for gym managers
and the franchise owner.

The interface is in Spanish (Mexico) and prices are in MXN. Code and documentation are
in English. "Gym Manager" is a working name; there is no final brand yet.

## Repositories

| Repository    | Purpose                                      |
| ------------- | -------------------------------------------- |
| `api-gym`     | Backend API (NestJS)                         |
| `gym-manager` | Frontend: a single PWA used by all roles     |
| `gym-docs`    | This documentation                           |

## Contents

- Roles
  - [Franchise Owner](roles/owner.md)
  - [Gym Manager](roles/manager.md)
  - [Reception](roles/reception.md)
  - [Member](roles/member.md)
- [MVP scope](mvp-scope.md)
- [Architecture](architecture.md)
- [Domain model](domain-model.md)
- [Decisions](decisions.md)

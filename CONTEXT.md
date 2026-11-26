# Eventra

Ubiquitous language of the internal operations dashboard for an event production agency. This
document defines the official domain glossary; technical and architectural decisions live in
`docs/adr/`.

## The business

**Agency**:
The organization that owns Eventra; coordinates between the clients requesting events and the
vendors who deliver event services.
_Avoid_: the company, we, firm.

**Client**:
The agency's customer; the organization or individual requesting an event.
_Avoid_: customer, brand, account, company.

**Vendor**:
An external service supplier in the agency's catalog (e.g., catering, audio, lighting, venue,
photography) assigned to deliver services for an event.
_Avoid_: provider, supplier, partner, contractor.

**Event**:
The system's unit of work: a client's event request, optionally assigned to a vendor and managed
by a producer, with a budget, event date, optional setup date, and lifecycle status.
_Avoid_: order, project, booking, gig, job.

## Roles

**User**:
An internal system user with credentials to sign in to Eventra.
_Avoid_: account, profile, member.

**Admin** (`admin`):
The role with full access: manages vendors, clients, users, and events, and is the only role
allowed to access restricted administrative pages and endpoints.
_Avoid_: administrator, superuser, root.

**Producer** (`producer`):
The standard operational role: manages events and reviews operational reports.
_Avoid_: coordinator, standard user, operator, manager.

## Lifecycle

**Event status**:
One and only one of these five stages, in lifecycle order: `inquiry`, `planned`, `confirmed`,
`in_progress`, `completed`.
_Avoid_: state, phase, stage, step.

**Seed data**:
The initial demo records inserted automatically by backend seeders when a database table is empty
on first launch.
_Avoid_: mock, fixture, test data, dummy data.

**Session**:
The authenticated state of a User while using Eventra (JWT token and loaded user profile).
_Avoid_: login (login is the act of signing in, not the active state).

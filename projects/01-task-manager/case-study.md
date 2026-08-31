# Task Manager API

**Stack** n8n, Airtable | **Status:** Production-tested

## The Problem

A team keeps tasks in Airtable but also wants a lightweight interface (or API) through which anyone — including non-Airtable users — can create, read, update and delete tasks without needing a seat in the tool. We need a full CRUD (Create, Read, Update, Delete/Archive) automation exposed via a simple interface, replacing manual data entry into the task tool.

## The Solution

A single authenticated webhook endpoint (`POST /tasks`) exposing full CRUD on an Airtable task
list, routed internally by an `action` field instead of separate endpoints per operation. A second,
unauthenticated `GET /tasks/info` endpoint documents the API contract for anyone integrating
against it. Create validates required fields and normalizes defaults; Read supports lookup by ID
or dynamic filtering by any combination of assignee/status/priority; Update maps only the fields
present in the request so partial updates never wipe unrelated data; Delete is a soft delete
(status flips to Archived, nothing is destroyed).

## Key Design Decisions

- **One webhook, action-based routing not four separate endpoints.** One URL to secure, one
  auth check, one thing to document. `{ "action": "create", ... }` vs. four different paths.
- **Validate before normalize.** No point defaulting optional fields on a submission that's about
  to get rejected for missing required ones.
- **Defaults applied in n8n, not relied on in Airtable.** Same result whether a record comes in
  through this API, gets entered directly in Airtable, or arrives from a future integration.
- **Update only sends fields present in the request.** A Code node filters the payload down to
  just what the client actually sent before it reaches Airtable, so `{ "status": "Done" }` can't
  silently blank out title/assignee/etc.
- **Soft delete, not hard delete.** Archived records are always recoverable; nothing is ever
  permanently destroyed by a bad request or a mis-click.
- **`id` lookups and filtered search are separate paths.** A single-ID lookup can
  meaningfully 404 ("not found"); a filtered search with no matches is just an empty list, not
  an error — different operations, kept as different branches.
- **`info` endpoint lives on its own unauthenticated webhook**, not routed through the main
  authenticated one — n8n enforces header auth at the trigger level, before any workflow logic
  runs, so a docs endpoint that skips auth has to be a genuinely separate trigger.

## Reliability & Edge Cases Handled

- **Missing required fields on create** - caught before any Airtable call, 400 with a clear
  message.
- **Non-existent `id` on read/update** - explicit 404, not a blank/broken response.
- **Empty update body** - rejected with "no fields to update" instead of a no-op write.
- **Unrecognized `action` value** - falls to a default branch, 400 "unknown action". No request
  disappears silently.
- **Unauthenticated requests** - rejected at the trigger level before any workflow logic runs.
- **Airtable write failures** (create/update/archive) → each has its own error branch returning a
  real error status, not a false-positive 200.

## Outcome / Impact

- Replaced manual Airtable data entry with a documented, testable API — full CRUD in one endpoint.
- Zero destructive deletes possible; every "delete" is reversible.
- Zero silent failures across the test suite — every branch (success or error) returns an explicit,
  correct status code and message.
- Self-documenting via the `/info` endpoint, so onboarding a new integration doesn't require a
  separate doc to stay in sync.

## Tech Stack

`n8n` `Airtable` `Webhook (dual trigger: authenticated + public)` `Header Auth (X-api-key)`
`Node types used: Webhook, Switch, IF, Code, Airtable (Create/Search/Update), Respond to Webhook`

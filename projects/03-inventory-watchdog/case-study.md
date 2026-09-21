# Inventory Reorder Watchdog — case-study.md

**Stack:** n8n, Airtable, Slack, Gmail | **Status:** Production-tested

## The Problem

A small retail/e-commerce operation only finds out stock is low when a customer complains it's
out. Nobody's watching inventory levels in real time, and nobody's doing a regular sweep either.
Both gaps needed covering: react instantly when a sale pushes something below or equal threshold,
and catch anything that slips through with a daily check.

## The Solution

Two separate workflows, not one. **Real-time reactive**: an order-placed webhook splits
multi-item orders, looks up each SKU in Airtable, validates that every SKU in the order actually
exists in inventory, calculates new stock (clamped at zero, flagged if the order oversold),
writes the update back, and alerts Slack, and also emails an internal reviewer if anything's now at or
below its reorder threshold. **Scheduled sweep**: a daily job that lists the whole inventory table,
filters for anything at or below threshold, and sends a single digest to Slack and Gmail. This workflow is
independent of whether the real-time flow already caught it, as a safety net.

## Key Design Decisions

- **Two workflows, not one** Different trigger types (webhook vs. schedule) doing genuinely
  different jobs (react vs. sweep).
- **Validate all SKUs in an order before processing any of them.** A Code node compares the
  original order's SKU list against what Airtable actually matched, so a typo'd or discontinued
  SKU gets caught and reported explicitly, instead of quietly producing no output and vanishing.
- **Oversold orders get a different message than routine low stock**, not just a clamped number.
  Running out because of one big order is a fulfillment problem happening right now; dipping
  below a reorder threshold is a "handle it this week" problem. Same underlying data
  (`is_oversold` flag), different urgency in the Slack message and the webhook response.
- **Reorder emails go to an internal reviewer, not the supplier.** No real purchase order gets
  auto-sent without a human looking at it first. The draft email says explicitly "no PO has been
  sent yet" so that's never ambiguous to whoever reads it later.
- **Continue On Fail on the inventory update step**, so one bad item in a multi-item order doesn't
  block the others from updating. Paired with an explicit error branch, a failed update gets
  logged, not silently dropped.

## Reliability & Edge Cases Handled

- **Oversell** (order quantity exceeds current stock); clamped to 0 with `Math.max(0, ...)`,
  flagged with an `is_oversold` boolean and a `shortfall` count, surfaced distinctly in both the
  Slack alert and the webhook response.
- **Unknown/invalid SKU in an order** caught by comparing the order's SKU list against what
  Airtable actually returned. First version of this branch had nowhere to route to — the mismatch
  was detected correctly but the alert node was never wired to anything, so it silently did
  nothing. Caught by deliberately testing with a bad SKU and noticing nothing showed up anywhere.
- **One item failing mid-batch shouldn't block the rest** Continue On Fail on the Airtable
  update, with the error output wired to its own alert instead of left dangling (same dangling-
  node mistake as above, fixed the same way).
- **Redundant filtering** — the scheduled sweep originally had a second IF node re-checking
  "is stock low?" on data that had already been filtered for exactly that. Removed it once
  noticed; one less thing to keep in sync with the actual filter logic.

## Outcome / Impact

- Stock drops below threshold surface in Slack within seconds of the triggering order, instead of
  being discovered when a customer complains.
- A daily digest catches anything the real-time path might miss, with zero manual checking.
- Two real "silent failure" bugs caught during testing - a dangling invalid-SKU branch and a
  dangling update-error branch - both fixed before they could hide a real problem in production.

## Tech Stack

`n8n` `Airtable` `Slack` `Gmail`
`Node types used: Webhook, Split In Batches, Airtable (Search/List/Update), Code, IF, Filter,
Schedule Trigger, Slack, Gmail, Respond to Webhook`

# Smart Support Ticket Router — user-guide.md

## What this does

Every new support ticket gets automatically read, categorized, and flagged by urgency — so
critical issues get noticed immediately instead of waiting in a queue with everything else.

## What triggers it

Every time a new ticket comes in through the support system's webhook.

## What you'll see when it works

- **Critical tickets** (outages, urgent billing issues on paying accounts) post to
  `#urgent-escalations` in Slack and send an email to on-call.
- **High priority tickets** (mostly Pro-tier customers with technical issues) post to `#high-response-team`.
- **Everything else** posts to `#normal-response-team`.
- Every single ticket, no matter where it's routed, gets a row added to the
  [Ticket Audit Log sheet] - ticket ID, category, urgency, where it was routed, and when.

## How to tell if something's wrong

- A ticket comes in but nothing shows up in Slack - check the workflow's Executions tab in n8n
  for an error on the Slack node.
- The audit log has a row but the urgency looks wrong - check the ticket's `plan_tier` field was
  included and spelled correctly (`Free`/`Pro`). This is what the urgency logic checks against.
- A ticket that should've been critical wasn't flagged - the classifier works off specific
  keywords (see below); if the wording didn't match anything in the list, it won't get flagged
  automatically. Report the exact wording so the keyword list can be updated.

## How urgency gets decided (plain-language version)

- **Critical:** This is for all Pro customers. The ticket mentions an outage (`down`, `outage`, `can't access`) - OR it's a
  billing issue using urgent language (`urgent`, `immediately`).
- **High:** anything from a Pro-tier customer that mentions the following keywords (error, `bug`,`crash`, `not working`, `glitch`, `account`, `password`,`login`, `register`, `sign up`, `sign in`).
- **Normal:** everything else.

## How to make small adjustments without touching n8n directly

- To change which words trigger a category (billing/technical/account) or which words count as
  urgent, someone with n8n access needs to update the keyword list in the "Category" or "Scale"
  node. This isn't something that can be changed from Slack or Google Sheets directly.
- To review what's been routed where, check the Ticket Audit Log sheet. It's the full history,
  searchable and sortable, independent of Slack.

## Who to contact if it breaks

You can send me a mail `oyebadesegunsam@gmail.com`

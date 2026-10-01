# E-commerce Order Fulfillment Bridge — user-guide.md

## What this does

When a paid order comes in, it automatically gets queued for fulfillment, updates (or creates)
the customer's record, and logs the revenue, all three at once, with no manual re-entry.

## What triggers it

Every time an order is placed through the store's order webhook.

## What you'll see when it works

- A new row in the **Pick & Pack** sheet, status "Awaiting Pick," ready for the warehouse.
- The customer's record in Airtable either created (first-time buyer) or updated (their order
  history and lifetime value grow).
- A new row in the **Revenue Ledger** sheet.
- A Slack message summarizing the order: `✅ Order ORD-90301 — ✅ Fulfillment, ✅ CRM, ✅ Ledger`.

## How to tell if something's wrong

- The Slack confirmation shows a ❌ next to one of the three systems (e.g.,
  `⚠️ Order ORD-90301 — ✅ Fulfillment, ❌ CRM, ✅ Ledger`): that system specifically didn't get
  updated for that order; check the automation error log and re-run that step manually if needed.
  The other two systems are confirmed fine — only the ❌'d one needs attention.
- An order doesn't show up anywhere at all: check whether `payment_status` on the original order
  was actually `"paid"` - unpaid/pending orders are logged separately (see below) and
  deliberately don't reach fulfillment, CRM, or the ledger.

## What happens to unpaid orders

They're logged to a separate **Unpaid Orders** sheet and a note goes to `#accounts-payable` -
they don't get fulfilled, added to the CRM, or counted as revenue until payment clears.

## Who to contact if it breaks

Contact the person who manages workflow. You can send me a mail also `oyebadesegunsam@gmail.com`

# Inventory Reorder Watchdog — user-guide.md

## What this does

Two automations that watch stock levels: one reacts the moment an order pushes something low,
the other checks the whole inventory once a day as a backup.

## What triggers them

- **Real-time:** every time an order comes in through the store's order-placed webhook.
- **Daily sweep:** automatically every morning at 7:00 AM, no action needed.

## What you'll see when it works

- A Slack alert the moment any item drops to or below its reorder threshold, including a special
  🚨 version if an order actually exceeded available stock (meaning a customer was sold something
  we don't have enough of).
- A draft reorder email to [internal reviewer] - this is a draft only, nothing gets sent to a
  supplier automatically. Review it and forward manually if the numbers look right.
- A daily digest in Slack and email each morning listing everything currently below threshold,
  whether or not it was already flagged during the day.

## How to tell if something's wrong

- An order comes in but no Slack alert, even though you'd expect one: check the item's SKU
  matches exactly what's in the Inventory table (typos won't match, and you'll instead see an
  "invalid SKU" alert for that order).
- The daily digest doesn't arrive one morning: check the workflow's Executions tab in n8n for a
  failed run around 7 AM.
- An item's stock update didn't apply after an order: check for an "update failed" alert in
  Slack - the system is built to flag this rather than fail silently, so if the number looks
  wrong, there should be a message explaining why.

## Who to contact if it breaks

Contact the person who manages workflow. You can send me a mail also `oyebadesegunsam@gmail.com`

# Resilient Invoice Processing Pipeline — user-guide.md

## What this does

Reads invoices that arrive by email, enters them into the invoice ledger, flags the large ones
for approval, and tells you in Slack what happened to each one.

## How to send an invoice in

Email it as a **PDF attachment** to the invoices inbox, with the `Invoices` label applied
automatically by the Gmail filter. Don't apply the label by hand to old emails after they've
arrived. Only new emails are picked up.

The PDF must be a **text-based PDF** (exported from accounting software or a document editor)
using this layout:

    INVOICE
    Invoice Number: INV-2026-0447
    Vendor: Acme Supplies Ltd
    Invoice Date: 2026-10-21
    Due Date: 2026-10-31
    Total Due: $1,450.00

**Not supported yet:** scanned invoices and photos of invoices. They'll be flagged for manual
entry.

## What you'll see in Slack

| Message                             | Meaning                                      | What to do                                                                     |
| ----------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------ |
| ✅ Invoice logged, auto-approved    | Amount was less than $1,000                  | Nothing                                                                        |
| 🟡 Approval needed                  | Amount is $1,000 or over                     | Review it, then change the status in the Invoices sheet                        |
| ⚠️ Couldn't read the file           | Scanned or unreadable PDF                    | Enter it manually                                                              |
| ⚠️ Missing or invalid fields        | The message names the fields                 | Ask the vendor for a corrected invoice                                         |
| 🔁 Possible duplicate               | That invoice number is already in the ledger | Check the existing row. If it's genuinely a new invoice, it needs a new number |
| 🚨 URGENT: couldn't write to ledger | The invoice was read but not saved           | Enter it manually, then tell whoever maintains the automation                  |

## About approvals

Invoices over or equals $1,000 stay as **Pending Approval** until a person changes the status.
Nothing approves them automatically.

## Where to check

- **Invoices sheet:** every invoice that was successfully processed.
- **Execution Log (Airtable):** one row per PDF received, with its status and the reason if it
  failed. If an invoice isn't in the ledger, the log says why.

## Who to contact if it breaks

Contact the person who manages workflow. You can send me a mail also `oyebadesegunsam@gmail.com`

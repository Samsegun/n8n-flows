# Resilient Invoice Processing Pipeline — reliability-runbook.md

For whoever is on the receiving end of an alert. Find the symptom, follow the action.

## Quick rules

- **One log row per PDF attachment.** Two PDFs in one email share a run ID and get two rows.
  Emails with no PDF produce no rows.
- **The invoice is only in the ledger if it succeeded.** A failed write leaves no ledger row,
  so re-sending is safe and won't be flagged as a duplicate.
- **To reprocess anything,** fix the cause, then send the email again as a new message.
  Existing emails are not re-read.

## Failure modes

| Symptom                                     | Log status              | Likely cause                                                                | What to do                                                              |
| ------------------------------------------- | ----------------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| "Couldn't read the file"                    | Extraction Failed       | Scanned/photographed PDF, or wrong layout                                   | Enter manually. Ask the vendor for a text-based PDF                     |
| "Missing or invalid fields: e.g. amount"    | Validation Failed       | A field is missing or unparseable                                           | Ask the vendor to correct and resend                                    |
| "Possible duplicate"                        | Duplicate               | Invoice number already in the ledger                                        | Check the existing row. Genuine resubmission needs a new number         |
| 🚨 URGENT, ledger write failed, took ~15s   | Write Failed            | Google Sheets unavailable (3 retries used up)                               | Check Google status, enter manually, resend once it's back              |
| 🚨 URGENT, ledger write failed, took ~3s    | Write Failed            | Configuration problem: tab renamed or deleted, sheet permissions changed    | Check the ledger tab name and sharing, fix, resend. Retrying won't help |
| Alert in `#automation-alerts` naming a node | (none, workflow failed) | Unexpected failure, e.g. the log write or a credential                      | Open the linked execution, read the error, fix the credential or setup  |
| Invoice emailed but nothing happened        | No execution            | Email had no PDF, wasn't labeled, or arrived before the workflow was active | Check the label and attachment, resend as a new email                   |
| Invoice stuck at Pending Approval           | (ledger status)         | Nobody has reviewed it                                                      | Review and change the status in the sheet                               |
| Slack message missing but ledger row exists | Success                 | Slack outage. Logging doesn't depend on Slack                               | Nothing is lost. Check the Execution Log                                |

## Monitoring checks

- **Log count vs attachments received.** If invoices arrived and the log has fewer rows, logging
  itself broke. Treat that as urgent.
- **Run duration.** A normal run takes 2 to 5 seconds. Around 15 seconds on a write means the
  retries fired, so something external is unhealthy.
- **Weekly:** skim for repeated Extraction Failed from the same vendor. It usually means that
  vendor's layout needs support.

# Form-to-Sheet Lead-Logger — How It Works

## What this does

This workflow collects a lead when someone fills out the website contact form. It checks that the email looks valid, saves the lead to the Google Sheet, sends a confirmation email, and posts a Slack alert so the team knows a new lead came in. If the email fails to send, the lead is still kept in the sheet so it can be retried (manually) later.

## What triggers it

The workflow starts every time someone submits the contact form on the website. The form data is sent to n8n, which validates it and begins the process.

## What you'll see when it works

- A new row appears in the Leads sheet within a few seconds
- The lead gets a confirmation email
- The row is updated to show that the email was successfully sent
- A Slack message is posted in the team channel

## How to tell if something's wrong

- No row appears in Google Sheets after a form submission
  This usually means the form data did not reach n8n, the workflow failed, or the sheet connection is not working. Check the n8n execution log and test the form again.

- The lead is in the sheet, but they did not get the confirmation email
  This usually means the Gmail step failed. Check Slack for the error alert and run the sheet-resend workflow for the unconfirmed lead or leads.

- A Slack alert never appears
  This usually means the Slack step failed or the channel or webhook is wrong. Review the workflow execution and confirm the Slack settings.

## Who to contact if it breaks

Contact the person who manages the website form or the automation workflow. If there is no owner assigned, check the n8n execution log, the Google Sheet, and Slack first, then fix the failed step or rerun the resend workflow.

## How to make small adjustments

- To change the lead tracking fields, update the columns in Google Sheets directly and update Form Submission node (1st node)
- To change the confirmation email text, edit the email template or message content used by the workflow
- To change who gets Slack alerts, update the Slack channel or message in the workflow
- To add or remove a field from the form, update the form itself and make sure the matching sheet columns still line up

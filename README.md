Deal Intake Automation

Automated pipeline that turns inbound pitch emails into structured CRM rows.

For each new email in a labelled Gmail folder, the system reads the pitch deck (or the email body if there is no deck), extracts the fields a VC cares about, checks whether the company is already in the CRM, writes a new row if it is not, generates a summary Google Doc, routes the deal to the right owner by sector, and posts a notification.

Built on Google Apps Script:  the CRM is a Google Sheet and the inbox is Gmail, so this runs natively against both with no external services to host or pay for. Field extraction uses the Gemini API returning structured JSON.


Setup
Time required: about 10 minutes. Steps 1- 3 are required; the rest are verification.
1. Make your own copy
Open the shared link and choose Make a copy. This copies the Sheet and the attached Apps Script project into your Drive. Everything then runs as you, against your Gmail and your Drive no access to my account is involved.

Open the script with Extensions → Apps Script.
2. Add a Gemini API key
The key is stored in Script Properties, not in the code, so it does not travel with the copy. You will need your own.

Go to aistudio.google.com → Get API key → Create API key. Free tier, no credit card.
In the Apps Script editor: Project Settings (gear icon) → Script Properties → Add script property
Property: LLM_API_KEY
Value: your key
Click Save script properties.

Verify by running the testKeyPresent function it prints the key's length and last four characters, never the key itself.
3. Create the Gmail label
In Gmail, create a label called deal-intake and apply it to the pitch emails you want processed. The script only reads threads carrying this label, so it can never touch the rest of your inbox.

Sample emails and a sample deck are in /samples if you want to reproduce the results below without sourcing your own.
4. Enable the Drive service
Should already be enabled in the copy. To confirm: in the editor sidebar, Services should list Drive API (v3). If not, click +, find Drive API, set version to v3, and add it. This is only used to convert .pptx decks to PDF.
5. Run the tests
Select runTests from the function dropdown and click Run.

First run triggers a permissions dialog. Choose your account → Advanced → Go to Dexter Intake (unsafe) → Allow. That warning appears because the script is unpublished, which is expected for an internal tool.

Expect 14 PASS lines in the execution log. These cover domain normalization and company-name matching and need no API key, no inbox, and no network.
6. Run it
Select processInbox and click Run. Check:

new rows in the Deals tab
one line per email in Run_Log
a summary Doc linked from each row

To run it on a schedule: clock icon in the sidebar → Add Trigger → function processInbox, event source Time-driven, Minutes timer, every 15 minutes.


Configuration
Two things are meant to be edited without touching code.

Sector_Owners tab:  the sector→owner mapping. This is the source of truth for routing, and the sector list is read into the LLM prompt at request time, so the model can only choose sectors that actually have an owner. Add a row and the model's vocabulary expands with it. Headers must stay lowercase (sector, owner_name, owner_email), though the code normalizes casing and spaces defensively.

Config.gs: everything else in one object: tab names, the Gmail search query, the fallback owner, the model name, the notification webhook URL, the free-email-domain list, and the company-name noise words.
Enabling notifications
CONFIG.WEBHOOK_URL is blank by default and the system logs a skip rather than failing. To enable Google Chat: open a space → space name dropdown → Apps & integrations → Webhooks → Add → paste the URL into CONFIG.WEBHOOK_URL. Slack incoming webhooks work with the same payload.

WhatsApp is a swap of the notify() function in Actions.gs and nothing else — Twilio needs a different URL, a form-encoded body, and an auth header, but no other file knows how notifications are sent. See the writeup for why this was stubbed rather than built.


File structure
File
Responsibility
Config.gs
All settings and the API key accessor
Sheets.gs
Header-keyed read/write and run logging
Dedupe.gs
Domain normalization and the duplicate decision
Extract.gs
Attachment handling, LLM call, response validation
Actions.gs
Sector routing, summary Doc, notification
Main.gs
Orchestration and per-email error isolation
Tests.gs
Fixture tests, runnable without inbox or API access


Nothing reads a column by number. Rows are keyed by header name, so inserting a column in the Sheet will not break the code.


Troubleshooting
LLM_API_KEY missing from Script Properties — step 2 was skipped, or the property name has a trailing space.



# n8n CRM ⇄ Google Sheets sync (HubSpot)

A simple, ready-to-import [n8n](https://n8n.io) workflow that keeps your **HubSpot contacts** and a **Google Sheet** in sync, in both directions.

| Direction | When | What it does |
|---|---|---|
| HubSpot → Google Sheets | Every hour | Contacts created or updated in HubSpot are added to the sheet, or their existing row is updated. Rows are matched by **email**. |
| Google Sheets → HubSpot | Every 5 minutes | New rows you type in the sheet (email filled, `hubspot_id` empty) are created or updated in HubSpot. The HubSpot ID is then written back to the row, so the same row is never sent twice. |

No credentials or personal data are included in this repository.

## Files

| File | Purpose |
|---|---|
| `crm-google-sheets-sync.json` | The n8n workflow to import |
| `contacts-template.csv` | The sheet layout (column headers + one example row) |

## Requirements

- An n8n instance (self-hosted or n8n Cloud), version 1.x or newer
- A HubSpot account (the free CRM is enough)
- A Google account with Google Sheets

## Setup

### 1. Create the Google Sheet

1. Create a new Google Sheet, for example **CRM Contacts**.
2. Rename the first tab to **Contacts**.
3. Paste these headers in row 1 (or import `contacts-template.csv` and delete the example row):

```
hubspot_id | email | first_name | last_name | phone | company | source | last_synced
```

The header names must be exactly the same, in lowercase.

### 2. Create a HubSpot private app token

1. In HubSpot, go to **Settings → Integrations → Private Apps → Create a private app**.
2. In **Scopes**, enable:
   - `crm.objects.contacts.read`
   - `crm.objects.contacts.write`
3. Create the app and copy the **access token**.

### 3. Import the workflow into n8n

1. In n8n, click **Workflows → Add workflow → … menu → Import from File** (or **Import from URL** and paste the raw GitHub link of `crm-google-sheets-sync.json`).
2. The workflow opens with two lanes: *HubSpot to Google Sheets* and *Google Sheets to HubSpot*.

### 4. Connect your accounts

Open each node with a warning icon and create or select its credential:

| Node | Credential type |
|---|---|
| Get recently updated HubSpot contacts | **HubSpot App Token** (paste the token from step 2) |
| Create or update HubSpot contact | the same HubSpot App Token |
| Add or update row in Google Sheets | **Google Sheets OAuth2** |
| Save HubSpot ID back to the sheet | the same Google Sheets OAuth2 |
| New row in Google Sheets | **Google Sheets Trigger OAuth2** |

Then, in the three Google Sheets nodes, pick your spreadsheet (**CRM Contacts**) and your tab (**Contacts**) from the lists.

### 5. Test, then activate

1. Click **Execute workflow** on the *Every hour* trigger: your recent HubSpot contacts should appear in the sheet.
2. Add a new row in the sheet with an email (leave `hubspot_id` empty), then execute the *New row in Google Sheets* trigger: the contact is created in HubSpot and its ID appears in the row.
3. When both tests pass, switch the workflow to **Active** (Publish).

## How it works

```
Every hour ─► Get recently updated HubSpot contacts ─► Map to sheet row ─► Keep rows with email ─► Add or update row (match: email)

New sheet row (every 5 min) ─► Only rows with email and no hubspot_id ─► Create or update HubSpot contact ─► Save HubSpot ID back to the row
```

- **Email is the unique key.** Emails are lowercased and trimmed on both sides to avoid duplicates.
- **No sync loop.** Rows written by the HubSpot → Sheets lane already have a `hubspot_id`, so the Sheets → HubSpot lane ignores them.
- The `source` column shows where a row came from (`hubspot` or `sheet`) and `last_synced` shows when it was last written.

## Customising

- **Change the frequency:** open the *Every hour* or *New row in Google Sheets* trigger and change the interval.
- **Sync more fields:** add the HubSpot property name in *Get recently updated HubSpot contacts → Properties*, add a column in the sheet, map it in *Map HubSpot contact to sheet row*, and add it to *Add or update row in Google Sheets*. For the other direction, add it in *Create or update HubSpot contact → Additional fields*.
- **Another CRM:** replace the two HubSpot nodes with the equivalent nodes of your CRM (Pipedrive, Zoho CRM, Salesforce, etc.) and keep email as the matching key.

## Limitations

- The HubSpot → Sheets lane uses HubSpot's *recently created or updated* endpoint, which covers recent changes only. For a first full import, temporarily change that node's operation to **Get Many** with **Return All**, run it once, then switch it back.
- Only **new** rows are sent to HubSpot. Editing an existing row in the sheet does not update HubSpot (by design, to keep the workflow simple and safe).
- Deleting a contact on one side does not delete it on the other.
- Google Sheets has API rate limits; this setup is designed for small and medium contact lists (up to a few thousand rows).

## License

[MIT](LICENSE)

## Author

Chafik Lounici — ITC Consulting

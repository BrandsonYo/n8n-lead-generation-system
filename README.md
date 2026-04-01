# n8n Lead Generation & Reply Handling System

Inbound SMS reply handler built in n8n. Receives webhook callbacks from TextBee when leads reply to outbound SMS, classifies the intent, logs the conversation to Google Sheets, and routes the lead to the right follow-up action.

---

## What it does

```
TextBee webhook (inbound SMS reply)
  → verify HMAC signature (security check)
  → match phone number to lead in Google Sheets
  → classify intent (Interested / Question / Call Back Later / Not Interested)
  → update lead status and reply count in sheet
  → send email notification for warm replies
  → trigger appropriate follow-up path
```

Handles the full inbound side of an SMS outreach campaign. Pairs with the [n8n SMS Outreach Pipeline](../n8n-sms-outreach-pipeline/) for end-to-end outreach automation.

---

## Stack

- **n8n** — automation platform
- **TextBee** — SMS gateway (webhook delivery)
- **Google Sheets** — lead CRM and conversation log
- **Gmail** — email notifications for warm leads

---

## Setup

### 1. Prerequisites

- n8n instance with a public webhook URL
- TextBee account with webhook configured to point at your n8n instance
- Google Sheets spreadsheet matching the outreach pipeline schema
- Gmail account for notifications

### 2. Credentials (set up in n8n UI)

| Credential name | Type |
|---|---|
| Google Sheets account | Google Sheets OAuth2 |
| Gmail account | Gmail OAuth2 |

### 3. TextBee webhook configuration

1. In TextBee dashboard: **Settings → Webhooks**
2. Set the webhook URL to: `https://your-n8n-instance.com/webhook/YOUR_WEBHOOK_PATH`
3. Copy your TextBee webhook secret and set it as `TEXTBEE_WEBHOOK_SECRET` in your environment

The workflow verifies the HMAC-SHA256 signature on every inbound request. Requests with invalid signatures are rejected immediately.

### 4. Environment variables

```bash
cp .env.example .env
```

| Variable | Where to find it |
|---|---|
| `TEXTBEE_WEBHOOK_SECRET` | TextBee dashboard → Webhooks → your secret |
| `GOOGLE_SHEETS_SPREADSHEET_ID` | Google Sheets URL |
| `NOTIFICATION_EMAIL` | The email address to receive warm lead alerts |

### 5. Import and configure

1. In n8n: **Workflows → Import from file → `workflow.json`**
2. Update credential IDs in Google Sheets nodes and Gmail nodes
3. Update the spreadsheet ID in all Google Sheets nodes
4. Update the webhook secret in the signature verification Code node
5. Update the notification email address in Gmail nodes
6. Activate the workflow and copy the webhook URL
7. Paste the webhook URL into TextBee

---

## Intent classification

Replies are classified into 4 buckets:

| Intent | Trigger phrases | Action |
|---|---|---|
| Interested | "yes", "interested", "tell me more" | Email alert + update status |
| Question | "how", "what", "?" | Email alert + update status |
| Call Back Later | "later", "busy", "next week" | Update status, re-queue |
| Not Interested | "no", "stop", "unsubscribe" | Opt out, update status |

Modify the classification logic in the routing nodes to match your use case.

---

## License

MIT

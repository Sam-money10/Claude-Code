# Make.com Automation Blueprints

Ready-to-import Make.com (Integromat) blueprints for automated lead management.

---

## Blueprint 1: Automated Missed Call Follow-Up — Instant Text Back

**File:** `missed-call-text-back-blueprint.json`

```
Missed Call → Instant SMS to Caller + Lead Logged to Google Sheets + Email Alert to You
```

**Three parallel actions fire the moment a missed call hits the webhook:**

| Route | Action | Purpose |
|-------|--------|---------|
| 1 | **Send SMS via Twilio** | Instant text-back so the lead knows you care |
| 2 | **Log to Google Sheets** | Phone, name, timestamp, status tracked automatically |
| 3 | **Email notification** | You get an HTML alert with all call details |

### Sample Text-Back Message

> Hi [Name], we noticed we just missed your call! We didn't want you to wait — how can we help you today?
>
> Feel free to reply to this text or let us know a good time to call you back.

If no caller name is available, it defaults to "Hi there."

---

## Blueprint 2: Lead Follow-Up After Quote — Automated Until Yes or No

A 3-scenario system that automatically follows up with leads after your business sends them a quote. Follows up on an escalating cadence until the lead gives a definitive **yes** or **no**.

### How It Works

```
Quote Sent → Lead Logged → Follow-Up #1 (Day 2) → #2 (Day 5) → #3 (Day 10) → #4 (Day 17) → #5 Final (Day 25)
                                    ↑                                                              ↓
                              Lead Replies ──→ Yes? → Mark WON, stop sequence, notify business
                                             → No?  → Mark LOST, stop sequence, notify business
                                             → Unclear? → Continue follow-ups, notify for review
```

### The Three Scenarios

| # | File | Trigger | Purpose |
|---|------|---------|---------|
| 1 | `lead-followup-intake-blueprint.json` | Webhook (when quote is sent) | Logs the lead to Google Sheets and starts the follow-up timer |
| 2 | `lead-followup-scheduler-blueprint.json` | Scheduled (daily at 9 AM) | Finds leads due for follow-up, sends the right message, updates the sheet |
| 3 | `lead-followup-response-blueprint.json` | Webhook (when lead replies) | Detects yes/no/undecided and routes accordingly |

### Follow-Up Cadence

| Follow-Up # | Day | Tone | Subject Line Style |
|-------------|-----|------|-------------------|
| 1 | Day 2 | Gentle check-in | "Following up on your quote" |
| 2 | Day 5 | Value & flexibility | "Quick question about your quote" |
| 3 | Day 10 | Soft nudge | "Still interested?" |
| 4 | Day 17 | Urgency / expiring | "Your quote is expiring soon" |
| 5 | Day 25 | Final break-up | "Closing the loop" |

Each follow-up is sent via **email** and optionally **SMS** (if the lead's phone number is on file).

### Lead Status Flow

```
pending → (follow-ups continue)
   ↓
won       ← lead said yes (sequence stops)
lost      ← lead said no (sequence stops)
no_response ← all 5 follow-ups sent with no reply (sequence stops)
```

### Yes/No Detection Keywords

The response handler detects definitive answers using keyword matching:

**YES indicators:** yes, let's do it, move forward, go ahead, sounds good, I'm in, sign me up, accept, approved, let's proceed, we're ready, book it, schedule it, ready to start, deal

**NO indicators:** no thank, not interested, pass, decline, not right now, not at this time, can't afford, too expensive, went with someone else, found another, decided against, no longer need, cancel, don't need, not for us, hard pass

**Undecided** (no match): Follow-ups continue, and the business owner gets notified to manually review the response.

---

## How to Import into Make.com

1. Go to [make.com](https://www.make.com) and log in
2. Click **Scenarios** → **Create a new scenario**
3. Click the **three dots (...)** menu at the bottom of the editor
4. Select **Import Blueprint**
5. Paste the contents of the blueprint JSON file
6. Click **Save**
7. Repeat for each scenario you want to use

---

## Setup Guide — Lead Follow-Up System

### Step 1: Create the Google Sheet

Create a Google Sheet called **"Lead Follow-Ups"** with these column headers in Row 1:

| A | B | C | D | E | F | G | H | I | J | K | L | M |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Lead Name | Lead Email | Lead Phone | Quote Amount | Quote Description | Quote Date | Status | Next Follow-Up Date | Follow-Up Count | Last Follow-Up Date | Business Name | Sender Name | Sender Email |

Copy the **Spreadsheet ID** from the URL (the long string between `/d/` and `/edit`).

### Step 2: Import All Three Scenarios

Import each blueprint file as a separate Make.com scenario:

1. **Intake** (`lead-followup-intake-blueprint.json`)
2. **Scheduler** (`lead-followup-scheduler-blueprint.json`)
3. **Response Handler** (`lead-followup-response-blueprint.json`)

### Step 3: Configure Connections

In **each** scenario, click on the modules and configure:

| Connection | Where | What to Do |
|-----------|-------|-----------|
| **Google Sheets** | All 3 scenarios | Connect your Google account, replace `YOUR_GOOGLE_SHEET_ID` |
| **Email (SMTP/Gmail)** | All 3 scenarios | Connect your email account for sending follow-ups and notifications |
| **Twilio** (optional) | Scheduler only | Add your Twilio credentials, replace `YOUR_TWILIO_PHONE_NUMBER` |
| **Notification email** | All 3 scenarios | Replace `YOUR_EMAIL@example.com` with your email |

### Step 4: Configure the Scheduler

For the **Scheduler** scenario:

1. Click the scheduling settings (clock icon)
2. Set it to run **daily** (or every few hours for faster follow-up)
3. Set the time (e.g., 9:00 AM in your timezone)

### Step 5: Connect Your Webhooks

**Intake Webhook** — Fire this when your business sends a quote:

```json
{
  "lead_name": "Jane Smith",
  "lead_email": "jane@example.com",
  "lead_phone": "+15551234567",
  "quote_amount": "$2,500",
  "quote_description": "Website redesign with SEO",
  "business_name": "Acme Web Co",
  "sender_name": "John",
  "sender_email": "john@acmewebco.com"
}
```

**Response Webhook** — Fire this when a lead replies:

```json
{
  "lead_email": "jane@example.com",
  "lead_phone": "+15551234567",
  "response_text": "Yes, let's move forward with the project!",
  "response_source": "email"
}
```

### Step 6: Connect Your Existing Tools

| Platform | How to Fire the Intake Webhook |
|----------|-------------------------------|
| **GoHighLevel** | Workflows → Trigger: Pipeline Stage Changed to "Quote Sent" → HTTP module → POST to webhook |
| **HubSpot** | Workflows → Trigger: Deal stage = "Quote Sent" → Webhook action → POST to webhook |
| **Salesforce** | Process Builder → Opportunity stage = "Proposal" → HTTP Callout to webhook |
| **QuickBooks** | Use Zapier/Make to watch for new estimates → POST to webhook |
| **FreshBooks** | Webhook on estimate creation → POST to webhook |
| **Manual / CRM** | Add a Zapier or Make button in your workflow to fire the webhook |

| Platform | How to Fire the Response Webhook |
|----------|--------------------------------|
| **Email (Gmail/Outlook)** | Create a Make scenario: Watch for emails matching the lead → POST to response webhook |
| **Twilio (SMS replies)** | Set Twilio incoming message webhook → POST to response webhook |
| **GoHighLevel** | Workflows → Trigger: Reply received → HTTP module → POST to response webhook |
| **Typeform / Google Forms** | On submission → POST to response webhook |
| **Manual** | Manually POST to webhook when you hear back from a lead |

### Required vs Optional Fields

**Intake Webhook:**

| Field | Required | Default if Missing |
|-------|----------|-------------------|
| `lead_name` | Yes | — |
| `lead_email` | Yes | — |
| `lead_phone` | No | Empty (SMS skipped) |
| `quote_amount` | No | "Not specified" |
| `quote_description` | No | Empty |
| `business_name` | No | "Our Company" |
| `sender_name` | No | "The Team" |
| `sender_email` | No | Empty |

**Response Webhook:**

| Field | Required | Default if Missing |
|-------|----------|-------------------|
| `lead_email` | Yes | — |
| `response_text` | Yes | — |
| `lead_phone` | No | Empty |
| `response_source` | No | Empty |

---

## Customization

### Changing the Follow-Up Cadence

Edit the **Scheduler** blueprint's route conditions and `next_followup_days` values:

- Module 10 (Follow-Up #1): Change `next_followup_days` from `3` to your preferred gap
- Module 11 (Follow-Up #2): Change `next_followup_days` from `5`
- Module 12 (Follow-Up #3): Change `next_followup_days` from `7`
- Module 13 (Follow-Up #4): Change `next_followup_days` from `8`

### Changing the Follow-Up Messages

Edit the `email_body`, `email_subject`, and `sms_body` fields in each route's Feeder module (modules 10-14 in the Scheduler).

Make.com expressions you can use:
- `{{1.A}}` — Lead name
- `{{1.B}}` — Lead email
- `{{1.D}}` — Quote amount
- `{{1.E}}` — Quote description
- `{{1.F}}` — Quote date
- `{{1.K}}` — Business name
- `{{1.L}}` — Sender name

### Adding More Follow-Ups

1. Add a new route in the Scheduler's router (module 3)
2. Set the condition to match the new follow-up count
3. Add a Feeder module with the message content
4. Update the previous route's `next_followup_days`

### Manually Stopping Follow-Ups

Change column G (Status) in the Google Sheet from `pending` to `won`, `lost`, or `no_response`. The scheduler will skip any lead that isn't `pending`.

---

## Troubleshooting

| Issue | Solution |
|-------|---------|
| Follow-ups not sending | Check that the Scheduler scenario is **turned ON** and scheduled to run. Verify the Google Sheet has leads with status `pending` and a past follow-up date. |
| Lead not found in response handler | Make sure `lead_email` matches exactly what's in column B of the sheet. |
| Wrong follow-up message sent | Check column I (Follow-Up Count) — it should match the route conditions (0, 1, 2, 3, 4). |
| SMS not sending | Verify Twilio connection and that the lead has a phone number in column C. |
| Yes/No not detected | The keyword list may not cover the exact phrasing. Either add keywords to module 2 in the Response Handler, or manually update column G in the sheet. |

---

## Cost Estimates

- **Make.com**: Free tier includes 1,000 operations/month (each scenario run = 1 operation per module)
- **Twilio SMS**: ~$0.0079 per outbound SMS in the US
- **Google Sheets**: Free
- **Email (SMTP)**: Free with Gmail/Outlook connection

# Automated Missed Call Follow-Up — Instant Text Back

A Make.com (Integromat) blueprint that automatically sends an SMS text-back when a call is missed, logs the lead, and notifies you via email.

## What It Does

```
Missed Call → Instant SMS to Caller + Lead Logged to Google Sheets + Email Alert to You
```

**Three parallel actions fire the moment a missed call hits the webhook:**

| Route | Action | Purpose |
|-------|--------|---------|
| 1 | **Send SMS via Twilio** | Instant text-back so the lead knows you care |
| 2 | **Log to Google Sheets** | Phone, name, timestamp, status tracked automatically |
| 3 | **Email notification** | You get an HTML alert with all call details |

## Sample Text-Back Message

> Hi [Name], we noticed we just missed your call! We didn't want you to wait — how can we help you today?
>
> Feel free to reply to this text or let us know a good time to call you back.

If no caller name is available, it defaults to "Hi there."

---

## How to Import into Make.com

1. Go to [make.com](https://www.make.com) and log in
2. Click **Scenarios** → **Create a new scenario**
3. Click the **three dots (⋯)** menu at the bottom of the editor
4. Select **Import Blueprint**
5. Paste the contents of `missed-call-text-back-blueprint.json`
6. Click **Save**

---

## Setup Checklist

After importing, you need to configure these connections:

### 1. Webhook (Trigger)
- Click the first module (Custom Webhook)
- Click **Add** to create a new webhook
- Copy the webhook URL — you'll paste this into your phone system (see below)

### 2. Twilio (SMS)
- Click the **Send SMS** module
- Add your Twilio connection (Account SID + Auth Token)
- Replace `YOUR_TWILIO_PHONE_NUMBER` with your Twilio number (format: `+1XXXXXXXXXX`)

### 3. Google Sheets (Lead Log)
- Click the **Log Lead to Google Sheet** module
- Connect your Google account
- Create a Google Sheet with these column headers in Row 1:

| A | B | C | D | E | F |
|---|---|---|---|---|---|
| Phone | Name | Date/Time | Call Status | Action Taken | Follow-Up Status |

- Replace `YOUR_GOOGLE_SHEET_ID` with your sheet ID (from the URL)

### 4. Email Notification
- Click the **Send Email** module
- Connect your email account
- Replace `YOUR_EMAIL@example.com` with your actual email

---

## Connecting Your Phone System

Send a POST request to your Make.com webhook URL with this JSON body whenever a call is missed:

```json
{
  "caller_phone": "+15551234567",
  "caller_name": "John Doe",
  "call_time": "2026-02-12T10:30:00Z",
  "call_status": "missed"
}
```

### Common Phone System Integrations

| Platform | How to Connect |
|----------|---------------|
| **Twilio** | Use a TwiML Bin or Studio Flow to POST to the webhook on `no-answer` |
| **GoHighLevel** | Workflows → Trigger: Call Status Changed → Filter: Missed → HTTP module → POST to webhook |
| **RingCentral** | Create a webhook subscription for `telephony/sessions` with `missed` status |
| **Vonage** | Set your event URL to the Make.com webhook |
| **Google Voice** | Use a Zapier/Make bridge (no native webhook) |
| **OpenPhone** | Settings → Webhooks → Add webhook for `call.completed` with `missed` status |
| **CallRail** | Settings → Webhooks → POST missed calls to your Make.com webhook |

### Minimum Required Field

Only `caller_phone` is required. All other fields are optional and will gracefully default:

| Field | Required | Default if missing |
|-------|----------|-------------------|
| `caller_phone` | Yes | — |
| `caller_name` | No | "Unknown Caller" |
| `call_time` | No | Current timestamp |
| `call_status` | No | "missed" |

---

## Customizing the Text Message

Edit the **Prepare SMS Message** module (module #3) to change the wording. The `sms_body` field supports Make.com expressions:

- `{{1.caller_name}}` — caller's name from the webhook
- `{{1.caller_phone}}` — caller's phone number
- `{{formatDate(now; "hh:mm A")}}` — current time formatted

---

## Troubleshooting

| Issue | Solution |
|-------|---------|
| Webhook not triggering | Verify your phone system is POSTing to the correct URL with `Content-Type: application/json` |
| SMS not sending | Check Twilio credentials, verify the "From" number is a valid Twilio number |
| Google Sheet not updating | Verify the Sheet ID and that column headers match |
| No email received | Check spam folder; verify the email connection in Make.com |

---

## Cost Estimates

- **Make.com**: Free tier includes 1,000 operations/month
- **Twilio SMS**: ~$0.0079 per outbound SMS in the US
- **Google Sheets**: Free
- **Email**: Free (with Gmail/Outlook connection)

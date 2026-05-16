# n8n-sightco-team-review

n8n workflows that automate sales reporting and team review for **SightCo Optical** — a fictional 5-person eyewear retailer I use as a reference scenario. The workflows are real; the company is made up.

---

## What this is

Most small retailers run their CRM on Google Sheets and a manager's Monday morning routine. These workflows replace parts of that routine with structured automation — n8n handles the orchestration, Claude handles the judgment calls (tier classification, summarization, enrichment), and Google Sheets stays as the source of truth.

The point isn't to be clever. It's to show that an SMB-grade workflow stack can be built without writing custom code, if you know how to wire the pieces together properly.

---

## What's in here

| File | What it does |
|---|---|
| `day10-performance-tier-logic.json` | Classifies sales staff into Top Performer / Solid / Needs Coaching based on revenue thresholds using a Switch node, and writes the tier back to the sheet |
| `p1-sightco-daily-sales-reporter.json` | Reads today's sales rows from Google Sheets, sends them to Claude for structured JSON summarization, formats a daily report, and delivers via WhatsApp (Twilio sandbox). Runs daily at 7am SGT via Schedule Trigger. |

Each file imports cleanly into any n8n instance.

---

## Project 1 — Daily Sales Reporter: how it works

Each node has one job. Separation of concerns means: change the format? Edit Format node only. Change recipient? Edit Twilio node only. Change Claude's prompt? Edit Stringify node only.

---

## Patterns worth pointing out

A few things in here that aren't obvious from a quick scan:

- **Two-Code-node API integration pattern** — Code BEFORE HTTP to build the request body in JavaScript (sidesteps n8n's body-mode quirks), Code AFTER HTTP to parse the response. Universal pattern for any future API integration.
- **Defensive JSON parsing** — LLMs occasionally return chatty preamble ("Looking at the data, here's the report: {...}") despite explicit instructions not to. Parsing finds the JSON inside whatever Claude returns via `indexOf('{')` and `lastIndexOf('}')`, rather than demanding perfect output.
- **Explicit upstream node references** — `$('Get row(s) in sheet').item.json.row_number` reaches back to the source of truth rather than relying on data carried through multiple intermediate nodes.
- **`$now.format('yyyy-MM-dd')`** for dynamic date filtering at runtime, swapped in only after the hardcoded version proved the filter mechanic worked.
- **IF + Stop & Error** to read Twilio's response and fail loudly on synchronous delivery errors (e.g., body validation failures, 24h window expiry). `error_code` is the diagnostic field.

---

## Known production constraints (documented honestly)

### WhatsApp 24-hour service window

Meta's WhatsApp Business policy: businesses can only send freeform messages to a user within 24 hours of that user's last inbound message. Outside that window, only Meta-approved Message Templates can be sent.

In sandbox: this fires as Twilio error code `63016`.

**For production deployment**, this workflow's `Send WhatsApp via Twilio` node should be replaced with:
- WhatsApp Business API account (not sandbox)
- Meta Business verification (3–7 days)
- Per-template approval from Meta

### Async delivery failures aren't caught synchronously

The current IF check reads errors that appear in Twilio's immediate HTTP response (e.g., 63016 window expiry, body validation failures). Errors that occur during async delivery — e.g., recipient not in sandbox (`63007`) — are returned later via Twilio's webhook callbacks, not the initial response.

**For production**, async delivery handling requires either:
- A polling loop (Wait node → HTTP Request to Twilio's Message Status URL)
- A separate n8n workflow that receives Twilio's status callback webhook

Both are intentionally out of scope for this learning project. Acknowledging the gap rather than hiding it.

---

## Running these yourself

1. Clone the repo
2. Install n8n (`npm install n8n -g`, or use Docker)
3. Import any `.json` workflow via n8n's import UI
4. Replace placeholders:
   - `YOUR_SHEET_ID_HERE` → your own Google Sheet ID
   - Phone numbers → your test phone (must be in Twilio sandbox)
5. Configure credentials:
   - Google Sheets OAuth2
   - Anthropic API key (Header Auth with `x-api-key`)
   - Twilio Basic Auth (Account SID + Auth Token)
6. Join the Twilio WhatsApp sandbox from your phone (`join <code>` to Twilio's sandbox number)
7. Click execute

---

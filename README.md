# n8n-sightco-team-review

n8n workflows that automate sales reporting, lead enrichment, and team review for **SightCo Optical** — a fictional eyewear retailer used as a reference scenario. The workflows are real; the company is made up.

---

## What's in here

| File | What it does |
|---|---|
| `performance-tier-logic.json` | Classifies sales staff into tiers based on revenue thresholds, writes the tier back to the sheet. |
| `p1-sightco-daily-sales-reporter.json` | Sheets → Claude (structured summary) → WhatsApp (Twilio). Runs daily at 7am SGT. |
| `p2-salesforce-lead-enricher.json` | Salesforce contact → Claude (firmographic inference from email domain) → write back to 4 custom fields. |

Each file imports cleanly into any n8n instance.

---

## Architecture

Same shape across both AI-powered workflows: **read → AI step → defensive parse → side effect**, with fail-loudly checks at the boundaries.

**P1 — Daily Sales Reporter**

Schedule → Get rows → Build body → Anthropic → Parse → Format → Twilio → IF: error_code empty? → Stop & Error if not

**P2 — Salesforce Lead Enricher**

Get contacts → IF: Id exists? → Build body → Anthropic → IF: content exists? → Parse → Update contact → IF: success? → Stop & Error on any "no"

---

## Patterns used

- **Two-Code-node API integration** — Code node before HTTP builds the body; Code node after parses defensively. Reusable for any LLM/API.
- **Defensive JSON parsing** — `indexOf('{')` / `lastIndexOf('}')` extracts JSON from whatever Claude returns, including chatty preamble.
- **Source-of-Truth Rule** — reach back to where data was born (`$('Get many contacts').item.json.Id`), not wherever it's been carried through.
- **Fail-loudly at canonical placements** — after a read (catch empty), after an API call (catch errors before the parser chokes), after a side effect (catch sync failures).
- **API acceptance ≠ real-world result** — HTTP 200 means request accepted, not action succeeded. Sync errors are caught; async confirmation is documented as a known gap.

---

## Known production constraints

**P1 — WhatsApp 24-hour service window.** Meta only allows freeform messages within 24h of a user's last inbound. Outside the window, Twilio returns error `63016`. Production requires a Meta-approved Message Template.

**P1 — Async delivery failures not caught synchronously.** IF check reads Twilio's immediate response. Async delivery failures (e.g. recipient not in sandbox, `63007`) require Twilio status callback webhooks. Out of scope.

**P2 — Enrichment is inferred, not verified.** Claude infers company/industry from the email domain. Demonstrates the pattern; production would use a data API (Clearbit, Apollo, ZoomInfo) for verified firmographics. Workflow shape unchanged; only the enrichment source changes.

**P2 — Synchronous-only write verification.** `Check Write Success` reads Salesforce's immediate response. Catches validation rules, deleted fields, FLS. Does NOT catch async processes (flows, triggers, downstream validation). Production would poll modified-date after Wait, or subscribe to Platform Events.

---

## Running these yourself

1. Clone the repo, install n8n (`npm install n8n -g` or Docker)
2. Import any `.json` via n8n's import UI
3. Replace placeholders (sheet IDs, phone numbers, Salesforce instance URL)
4. Configure credentials in n8n: Google Sheets (OAuth2), Anthropic (Header Auth), Twilio (Basic Auth), Salesforce (OAuth2 via External Client App)
5. For P1: join the Twilio WhatsApp sandbox (`join <code>` to the sandbox number)
6. For P2: create 4 custom fields on Contact (`Inferred_Company__c`, `Inferred_Industry__c`, `Company_Size_Guess__c`, `Outreach_Angle__c`)
7. Execute

---

## What's next

- [x] Project 1 — Daily Sales Reporter
- [x] Project 2 — Salesforce Lead Enricher
- [ ] Project 3 — RAG-powered sales knowledge bot (Supabase vector store + Claude)

---

*Last updated: 29 May 2026*

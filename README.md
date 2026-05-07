# n8n-sightco-team-review

n8n workflows that automate sales performance reviews, lead enrichment, and customer ops for **SightCo Optical** — a fictional 5-person eyewear retailer I use as a reference scenario. The workflows are real; the company is made up.

---

## What this is

Most small retailers run their CRM on Google Sheets and a manager's Monday morning routine. These workflows replace parts of that routine with structured automation — n8n handles the orchestration, Claude handles the judgment calls (tier classification, enrichment, summarisation), and Google Sheets stays as the source of truth.

The point isn't to be clever. It's to show that an SMB-grade CRM stack can be built without writing custom code, and without paying for Salesforce, if you know how to wire the pieces together properly.

---

## What's in here

| File | What it does |
|---|---|
| `day7-webhook-trigger.json` | Webhook entry point — lets external systems push data into the pipeline |
| `day8-if-switch-routing.json` | Routes incoming items down different branches based on category |
| `day9-sheets-claude-enrichment.json` | Reads a Google Sheet, enriches each row via Claude, writes results back |
| `day10-performance-tier-logic.json` | Classifies sales staff into Top Performer / Solid / Needs Coaching based on revenue thresholds, writes the tier back to the sheet |

Each file imports cleanly into any n8n instance.

---

## Patterns worth pointing out

A few things in here that aren't obvious from a quick scan:

- **Set node before HTTP Request** — converts numbers to strings so values don't silently drop when sent to Claude
- **Switch node for 3-way classification** — cleaner than nested IFs once you have more than two branches
- **`$('Get row(s) in sheet').item.json.row_number`** — explicit upstream reference so Update Row knows which row to write back to, even after the data passes through several nodes
- **Loop Over Items** for row-by-row processing instead of trying to handle batches in a single API call

---

## Running these yourself

1. Clone the repo
2. Install n8n (`npm install n8n -g`, or use Docker)
3. Import any `.json` workflow via n8n's import UI
4. Plug in your own credentials — Google Sheets OAuth2 and an Anthropic API key
5. Swap the SightCo sample sheet ID for your own
6. Click execute

---

## About me

I'm Edmund. I work in optical retail in Singapore and I'm working my way into CRM and Revenue Operations — specifically the bit where AI actually has to do something useful, not just sit in a slide deck.

Right now I'm studying for the Salesforce Administrator certification, learning n8n in the slot between work and dinner, and building things like this repo to prove I can do the work, not just talk about it. Long-term plan is to land in CRM consulting and eventually relocate to New York.

If you're hiring for CRM/RevOps roles where AI integration matters, I'd be happy to talk.

---

## What's next

- [ ] Day 11 — Structured JSON output from Claude
- [ ] Day 12 — Gmail node for automated outreach
- [ ] Day 13 — Schedule Trigger for nightly batches
- [ ] Day 18 — Capstone: AI Lead Enricher (n8n + Claude + Salesforce)
- [ ] Phase 2 — RAG-powered sales knowledge bot (Supabase vector store)

---

*Last updated: May 2026*

# n8n-sightco-team-review

An n8n workflow that automates sales team performance reviews for **SightCo Optical** — a fictional 10-person eyewear retailer I use as a reference scenario. The workflow is real; the company is made up.

---

## What this is

Most small retailers run their CRM on Google Sheets and a manager's Monday morning routine. This workflow replaces part of that routine with structured automation — n8n handles the orchestration, multi-way logic classifies each salesperson into a performance tier, and the result gets written back to the source sheet.

The point isn't to be clever. It's to show that an SMB-grade workflow can be built without writing custom code, if you know how to wire the pieces together properly.

---

## What's in here

| File | What it does |
|---|---|
| `day10-performance-tier-logic.json` | Reads sales staff revenue from a Google Sheet, classifies each row into Top Performer / Solid / Needs Coaching using a Switch node, and writes the tier back to the sheet |

The workflow imports cleanly into any n8n instance.

---

## How the classification works

Three tiers based on monthly revenue thresholds:

| Tier | Revenue threshold |
|---|---|
| Top Performer | ≥ $50,000 |
| Solid | $30,000 – $49,999 |
| Needs Coaching | < $30,000 |

A Switch node evaluates each row top-to-bottom. The first matching rule wins. The result is then stamped onto the row and written back via Update Row in Sheet.

---

## Patterns worth pointing out

A few things in here that aren't obvious from a quick scan:

- **Set node before HTTP-style downstream work** — converts numbers to strings so values don't silently drop in later nodes
- **Switch node for 3-way classification** — cleaner than nested IFs once you have more than two branches
- **`$('Get row(s) in sheet').item.json.row_number`** — explicit upstream reference so Update Row knows which row to write back to, even after the data passes through Switch and Set nodes
- **Loop Over Items** for row-by-row processing — keeps the logic readable and lets the Switch evaluate each row independently

---

## Running it yourself

1. Clone the repo
2. Install n8n (`npm install n8n -g`, or use Docker)
3. Import `day10-performance-tier-logic.json` via n8n's import UI
4. Plug in your own credentials — Google Sheets OAuth2
5. Swap the sample sheet ID for your own (with columns: `name`, `email`, `revenue`, `performance_tier`)
6. Click execute

---

## About me

I'm Edmund. I work in optical retail in Singapore and I'm working my way into CRM and Revenue Operations — specifically the bit where AI actually has to do something useful, not just sit in a slide deck.

Right now I'm studying for the Salesforce Administrator certification, learning n8n in the slot between work and dinner, and building things like this repo to prove I can do the work, not just talk about it. Long-term plan is to land in CRM based roles.

If you're hiring for CRM/RevOps roles where AI integration matters, I'd be happy to talk.

---

## What's next

More workflows will be added as I build them. Roughly in order:

- [ ] Structured JSON output from Claude
- [ ] Gmail node for automated outreach
- [ ] Schedule Trigger for nightly batches
- [ ] AI Lead Enricher capstone (n8n + Claude + Salesforce)
- [ ] RAG-powered sales knowledge bot (Supabase vector store)

---


## Local Development

This repo is now managed locally with Git from the terminal & will undergo edits on weekly basis.


*Last updated: May 2026*

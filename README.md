# Automation & QA Developer — Take-Home Skills Assessment

**Candidate:** Srimanth  
**Submission Date:** May 21, 2026  
**Role Applied:** Automation & QA Developer

---

## Project Overview

This repository contains all three deliverables for the take-home assessment:

| Task | Description | Location |
|---|---|---|
| Task 1 | Web App QA & Debug Report (8 bugs) | [`Task1_QA_Report/`](./Task1_QA_Report/) |
| Task 2 | n8n GitHub Digest Workflow | [`Task2_n8n_Workflow/`](./Task2_n8n_Workflow/) |
| Bonus | Uptime Monitor Workflow | [`Bonus_UptimeMonitor/`](./Bonus_UptimeMonitor/) |

---

## Task 1 — Web App QA & Debug Report

**App tested:** [Conduit (demo.realworld.io)](https://demo.realworld.io) — an open-source Medium.com clone built with Angular + Node.js

**Testing approach:** Manual exploratory testing across all main user flows:
- Sign-up, Login, Logout
- Create / Edit / Delete articles
- Add / Remove tags
- Post / view comments
- Profile settings
- Feed filtering (Global vs Your Feed)

**Bugs found:** 8 issues documented, ranging from Critical (JWT in localStorage = XSS risk) to Low (no comment length limit).

**Highlights:**
- **Critical Bug #4:** JWT token stored in `localStorage` exposes users to session hijacking via XSS
- **High Bug #1:** Duplicate username error shows as generic top-of-form error instead of inline field feedback
- **High Bug #3:** Article editor allows empty title/body — publishes blank articles with no validation
- Detailed root-cause analysis written for Bug #4 (the localStorage/XSS issue)

📄 [Read the full report](./Task1_QA_Report/Task1_QA_Report_Srimanth.md)

---

## Task 2 — n8n API Integration Workflow

**File:** `Task2_n8n_Workflow/Task2_Workflow_Srimanth.json`

### What it does:
1. **Triggers** every 1 hour via n8n Schedule Trigger
2. **Fetches** top 10 JavaScript repos from GitHub Search API (sorted by stars)
3. **Transforms** results — keeps top 5, maps to clean flat objects with human-readable dates
4. **Enriches** each repo by calling the GitHub Repo Details API for additional metadata (watchers, license, archived status)
5. **Branches** via IF node: repos with > 1,000 ⭐ get a "high stars" embed; others get an "emerging repos" embed
6. **Posts** formatted Discord embed messages to a Discord webhook
7. **Error handling:** Global Error Trigger node catches any failure and posts a red alert to Discord — no silent failures

**APIs used:**
- `api.github.com/search/repositories` — primary data source (free, no auth needed)
- `api.github.com/repos/{owner}/{repo}` — enrichment (same API, second endpoint)
- Discord Incoming Webhook — output channel

📄 [Read the workflow README](./Task2_n8n_Workflow/README.md)

---

## Bonus — Uptime Monitor Workflow

**File:** `Bonus_UptimeMonitor/Bonus_UptimeMonitor_Srimanth.json`

### What it does:
1. **Pings** `demo.realworld.io` every 5 minutes
2. **Checks** HTTP status code — expects `200 OK`
3. **Retry logic:** If first ping returns non-200, waits and retries once before alerting
4. **Alerts Discord** if the site is confirmed down (2 consecutive failures)
5. **Slow response warning:** If response time > 3,000ms (even on a 200 OK), sends an amber warning embed
6. **Daily summary** at 9 AM every day with instructions to review n8n execution history
7. **Error handler:** Catches workflow-level errors (network, n8n issues) and alerts Discord

---

## Setup Notes

### Prerequisites
- n8n instance running (Docker: `docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n` OR `npx n8n`)
- A Discord server with an Incoming Webhook configured

### Credentials Setup
In n8n, create one credential:
- **Type:** HTTP Header Auth
- **Name:** `Discord Webhook Credentials`
- **Header Name:** `discordWebhookUrl`
- **Value:** Your full Discord webhook URL

### Import Order
1. Import `Task2_Workflow_Srimanth.json` first
2. Import `Bonus_UptimeMonitor_Srimanth.json`
3. Set the Discord credential on all HTTP Request nodes
4. Activate both workflows

---

## Key Design Decisions

| Decision | Rationale |
|---|---|
| GitHub API (no auth) | Public rate limit (10 req/min) is sufficient for this workflow; avoids credential complexity for reviewers |
| Discord webhook output | Free, instant setup, rich embeds, widely used — easiest for evaluator to verify |
| Stars > 1,000 threshold | Represents meaningful community adoption vs. emerging/niche projects; creates genuinely different content in both branches |
| Error Trigger (not try/catch in Code node) | n8n's built-in Error Trigger handles ALL node types uniformly, including HTTP timeout — more robust than per-node error handling |
| Retry logic in Uptime Monitor | Single-ping alerting generates too many false positives (DNS blip, transient timeout); two consecutive failures = real outage |

---

*All work is original. AI tools were used for drafting and reviewing — all decisions can be explained and defended.*

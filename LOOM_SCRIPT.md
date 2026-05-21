# 🎥 Loom Video Script — QA Assessment Walkthrough
**Estimated recording time: 8–12 minutes**

---

## [0:00 – 0:30] Intro

> "Hi, I'm Srimanth, and this is my submission for the Automation & QA Developer take-home assessment. I'll walk you through all three tasks — the QA bug report, the n8n API integration workflow, and the bonus uptime monitor. Let's jump in."

**[Show GitHub repo: https://github.com/Srimanth3272/software-]**

> "Everything is in this public GitHub repo. You can see the folder structure — Task1, Task2, and Bonus — each with their own deliverables. Let me open each one."

---

## [0:30 – 4:00] Task 1 — QA Bug Report

**[Open Task1_QA_Report_Srimanth.md in browser or VS Code]**

> "For Task 1, I tested the Conduit RealWorld demo app at demo.realworld.io — it's a Medium.com clone built with Angular on the frontend and Node.js on the backend."

> "I went through all the main user flows — sign-up, login, creating and editing articles, posting comments, and the settings page. I found 8 issues in total."

**[Scroll through the bug table slowly]**

> "Let me highlight a few key ones:"

**Bug #4 — Critical:**
> "The most serious issue is Bug 4 — the JWT token is stored in localStorage. This is a classic security anti-pattern. If there's any XSS vulnerability on the page — and in an app that renders user-submitted Markdown, that's a real risk — an attacker can steal the token with a single line of JavaScript and fully hijack the user's session."

**Bug #3 — High:**
> "Bug 3 is that the article editor accepts completely empty titles and bodies. There's no client-side validation and no server-side enforcement either. You can publish a blank article that shows up in the feed as an empty clickable link."

**Bug #1 — High:**
> "Bug 1 is more of a UX issue — when you try to register with a username that's already taken, the API does return the right error, but the frontend just dumps it as a generic error block at the top of the form. There's no field-level highlighting, so users don't immediately know which field caused the problem."

**[Scroll to Root Cause Analysis section]**

> "For the root-cause analysis, I picked the localStorage JWT issue because it's the highest severity and has a clear, well-understood fix. I walk through exactly what's happening, why it's exploitable, and the complete remediation — HttpOnly cookies, Content Security Policy, and token rotation."

---

## [4:00 – 7:30] Task 2 — n8n GitHub Digest Workflow

**[Open Task2_n8n_Workflow folder, show the JSON and README]**

> "For Task 2, I built an n8n workflow that fetches trending JavaScript repos from the GitHub Search API, enriches them with a second API call, and posts a formatted digest to Discord every hour."

**[Open Task2_n8n_Workflow/screenshots/workflow_canvas.png]**

> "Here's the workflow canvas diagram. Let me walk through each section:"

**Explaining the flow:**
> "It starts with a Schedule Trigger — fires every hour. Then it calls the GitHub Search API for JavaScript repos sorted by stars. A Code node filters this down to the top 5 and maps them to clean flat objects with human-readable dates and topics."

> "Then for each of those 5 repos, it calls the GitHub Repo Details endpoint — that's the enrichment step — to get extra metadata like the license, number of watchers, and whether the repo is archived."

> "Next is the IF node — if a repo has more than 1,000 stars, it routes to the 'established repos' Discord message. If it's under 1,000, it routes to the 'emerging repos' message. Both send to Discord but with different colors and formatting."

**Error handling:**
> "The key error handling is the Error Trigger node at the bottom. Any node failure anywhere in the workflow — a timeout, a bad API response, Discord being down — gets caught here and immediately posted as a red alert to Discord. No silent failures."

> "Credentials are stored in n8n's Credentials store and referenced by name — no hardcoded secrets anywhere in the workflow JSON."

---

## [7:30 – 10:00] Bonus — Uptime Monitor

**[Open Bonus_UptimeMonitor folder]**

**[Open screenshots/workflow_canvas.png]**

> "The bonus uptime monitor pings demo.realworld.io every 5 minutes. But I didn't want it to alert on the first failed ping — transient DNS blips and network timeouts are common. So I added retry logic: if the first ping returns non-200, the workflow retries once. Only if both pings fail does it send a red 'CONFIRMED DOWN' alert to Discord."

> "I also added response-time tracking. Even if the app returns 200, if the response takes more than 3 seconds, it sends an amber 'Slow Response Warning' — so you catch performance degradation before it becomes an outage."

> "There's also a daily 9 AM summary trigger that sends a digest to Discord with instructions to check n8n's execution history for a full uptime log."

> "And just like Task 2, there's an Error Trigger for workflow-level failures."

---

## [10:00 – 11:00] Wrap-up

> "So to summarize what I've delivered:"
> - "Task 1: 8 documented bugs on demo.realworld.io with a detailed root-cause analysis on the critical JWT security issue"
> - "Task 2: A fully functional n8n workflow using two GitHub API endpoints, an IF branch on a meaningful threshold, Discord output, and a global error handler"
> - "Bonus: An uptime monitor with retry logic, response-time alerts, daily summaries, and no silent failures"

> "Everything is in the GitHub repo linked in the submission form. I'm happy to walk through any part of this in more detail during the follow-up interview. Thanks for reviewing!"

---

## 📋 Quick Tips for Recording
- Record your screen showing the GitHub repo while narrating
- For Task 2/Bonus: show the workflow canvas images and the JSON file open in VS Code
- Keep it conversational — you don't need to read verbatim
- Aim for 8–10 minutes total

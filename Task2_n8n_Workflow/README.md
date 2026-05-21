# Task 2 — n8n GitHub Digest Workflow

**Author:** Srimanth  
**Workflow file:** `Task2_Workflow_Srimanth.json`

---

## APIs Used & Why

### Primary API: GitHub Search API
- **Endpoint:** `GET https://api.github.com/search/repositories?q=topic:javascript&sort=stars&order=desc&per_page=10`
- **Why:** GitHub's search API is free, requires no OAuth for public data, returns rich metadata (stars, forks, language, topics, license), and has generous rate limits (10 req/min unauthenticated). The JavaScript ecosystem is large and active, making it ideal for a meaningful daily digest.

### Enrichment API: GitHub Repository Details
- **Endpoint:** `GET https://api.github.com/repos/{owner}/{repo}`
- **Why:** Adds fields not available in search results — specifically `subscribers_count` (watchers), `network_count` (fork tree size), `license.spdx_id`, `default_branch`, and `archived` status. This gives a richer picture of repository health beyond just star count.

### Output: Discord Webhook
- **Why:** Free, instant to set up (no additional accounts needed), supports rich embeds with colors/formatting, and is widely used by developer teams. No SDK needed — just a `POST` with JSON.

---

## Workflow Structure

```
Schedule (every 1h)
  └── GitHub Search API (top 10 JS repos by stars)
        └── Code Node: Filter & Transform → Top 5 repos
              └── GitHub Repo Details API (enrichment for each repo)
                    └── Code Node: Merge enriched data
                          └── IF Node: stars > 1,000?
                                ├── TRUE  → Format "High Stars" Embed → Discord
                                └── FALSE → Format "Low Stars" Embed  → Discord

Error Trigger (global)
  └── Format Error Embed → Discord (fallback error channel)
```

---

## Transformation Logic

The **Code node** (`Transform - Filter Top 5 Repos`) does:
1. Reads `data.items` from the GitHub Search API response
2. Slices to the first 5 items (top 5 by stars)
3. Maps each to a clean flat object: `rank`, `name`, `description`, `stars`, `forks`, `language`, `url`, `openIssues`, `updatedAt` (human-readable), `topics` (comma-separated, max 3)
4. Returns the array as individual n8n items so downstream nodes process each repo independently

---

## Conditional Branch Logic

The **IF node** (`IF Stars > 1000`) routes items:
- **True branch (stars > 1,000):** Repo is "established" — formatted with bold star count, detailed embed with all enriched fields, posted to Discord with a blue embed color
- **False branch (stars ≤ 1,000):** Repo is "emerging" — formatted with a simpler embed and an amber color, labeled "emerging or niche"

**Threshold rationale:** 1,000 stars represents a meaningful signal of community adoption vs. a genuinely new/niche project. This threshold creates actionably different content in both branches.

---

## Error Path Behavior

The workflow uses n8n's **Error Trigger** node:
1. If **any** node in the workflow fails (API timeout, malformed response, Discord webhook down), n8n automatically routes to the Error Trigger
2. The `Format Error Alert` Code node structures a red Discord embed with: workflow name, failed node name, error message, and timestamp
3. The error embed is posted to the **same Discord webhook** (a separate dedicated error-notifications channel is recommended in production)
4. **No silent failure:** Every failure generates a visible alert. The workflow will NOT produce a missing digest without a corresponding error notification.

---

## Setup Instructions

### 1. Import Workflow
1. Open n8n → Workflows → Import from File
2. Select `Task2_Workflow_Srimanth.json`

### 2. Configure Discord Credential
1. n8n → Credentials → New → HTTP Header Auth
2. Name it `Discord Webhook Credentials`
3. Set `discordWebhookUrl` to your Discord webhook URL (Server Settings → Integrations → Webhooks → Copy URL)

### 3. Activate & Test
1. Click **Execute Workflow** to run manually once
2. Verify Discord receives the embed
3. Toggle **Active** to enable the hourly schedule

### 4. Testing with curl (Webhook variant)
If you switch the trigger to a Webhook node, test with:
```bash
curl -X POST https://your-n8n-instance/webhook/github-digest
```

---

## Notes
- GitHub's unauthenticated rate limit is 10 req/min. With 5 repos enriched per run (5 API calls) plus 1 search call = 6 calls/hour, well within limits.
- To add GitHub auth and increase rate limits: add a GitHub credential in n8n and add the `Authorization: token <PAT>` header.

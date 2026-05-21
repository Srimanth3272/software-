# n8n Setup Guide — Step-by-Step

## Prerequisites
- Node.js 18+ installed (check: `node --version`)
- OR Docker Desktop installed

---

## Step 1 — Install & Start n8n

### Option A: Using npx (Recommended — No Docker needed)
```bash
npx n8n
```
Wait for: `Editor is now accessible via: http://localhost:5678`

### Option B: Using Docker
```bash
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
```

Open your browser → **http://localhost:5678**

---

## Step 2 — Create Discord Webhook

1. Open Discord → Go to any server you own (or create a free test server)
2. Right-click a text channel → **Edit Channel**
3. **Integrations** tab → **Webhooks** → **New Webhook**
4. Give it a name (e.g., `n8n-digest`)
5. Click **Copy Webhook URL** — save this, you'll need it in Step 4

---

## Step 3 — Import the Workflows

### Import Task 2 Workflow
1. In n8n: click **Workflows** in the left sidebar
2. Click **Add Workflow** → **Import from File**
3. Select: `Task2_n8n_Workflow/Task2_Workflow_Srimanth.json`
4. Click **Import**

### Import Bonus Uptime Monitor
1. Repeat: **Add Workflow** → **Import from File**
2. Select: `Bonus_UptimeMonitor/Bonus_UptimeMonitor_Srimanth.json`
3. Click **Import**

---

## Step 4 — Set Up Discord Credentials

> ⚠️ You must do this **before** activating the workflows.

1. In n8n: go to **Settings** → **Credentials** → **Add Credential**
2. Search for: **HTTP Header Auth**
3. Fill in:
   - **Name:** `Discord Webhook Credentials`
   - **Name (header field):** `discordWebhookUrl`
   - **Value:** *(paste your Discord webhook URL from Step 2)*
4. Click **Save**

---

## Step 5 — Assign Credentials to Nodes

For **each** HTTP Request node that posts to Discord (in both workflows):

1. Open the workflow
2. Click on the Discord HTTP Request node
3. Under **Authentication** → select `Discord Webhook Credentials`
4. Repeat for all Discord nodes

> **Nodes to update in Task 2:**
> - `Discord - Post High Stars Digest`
> - `Discord - Post Low Stars Digest`
> - `Discord - Post Error Alert`

> **Nodes to update in Bonus:**
> - `Discord - Send Down Alert`
> - `Discord - Send Slow Alert`
> - `Discord - Send Daily Summary`
> - `Discord - Workflow Error Alert`

---

## Step 6 — Test Manually

### Test Task 2
1. Open `Task2_GitHub_Daily_Digest_Srimanth` workflow
2. Click **Execute Workflow** (▶ button, top right)
3. Watch the nodes light up green ✅
4. Check Discord — you should see a rich embed with top JS repos

### Test Bonus Uptime Monitor
1. Open `Bonus_UptimeMonitor_Srimanth` workflow
2. Click **Execute Workflow**
3. Check Discord — if demo.realworld.io responds with 200, no alert fires (normal)
4. To test alerts: temporarily change `targetUrl` in the `Initialize Check` node to `https://this-url-does-not-exist-xyz.com` → Execute → Discord alert fires

---

## Step 7 — Activate Both Workflows

1. Open each workflow
2. Toggle the **Active** switch (top right) to ON
3. Both will now run on their schedules:
   - Task 2: every 1 hour
   - Uptime Monitor: every 5 minutes + daily 9 AM summary

---

## Capturing Screenshots for Submission

After a successful test execution:
1. **Canvas screenshot:** Press `Ctrl+Shift+S` in n8n OR take a fullscreen screenshot of the workflow canvas
2. **Execution screenshot:** Go to **Executions** tab → click the latest execution → screenshot the green node outputs

---

## Troubleshooting

| Issue | Fix |
|---|---|
| `401 Unauthorized` on GitHub | Add `Authorization: token YOUR_PAT` header to GitHub nodes |
| Discord returns `400 Bad Request` | Check that the body is valid JSON; verify webhook URL is correct |
| `ECONNREFUSED` on uptime ping | The target app may be temporarily down — this is expected behavior |
| Nodes show red after import | Reassign credentials (Step 5) — credentials don't transfer with JSON exports |

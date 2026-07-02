# 🔧 CommitLog — Full Setup Guide

This guide walks through every credential and permission needed to run CommitLog end-to-end.

## Prerequisites

- An [n8n](https://n8n.io) account (Cloud or self-hosted)
- A GitHub account with a repo to track
- A Google account (for Gemini + Gmail)
- A LinkedIn account

---

## 1. GitHub Webhook

1. Go to your repo → **Settings → Webhooks → Add webhook**
2. Payload URL: your n8n webhook's **Production URL** (not the Test URL — that only works while actively "listening" in the editor)
3. Content type: **application/json** (not the default form-urlencoded — this matters, expressions expect proper JSON)
4. Which events: **Just the push event**
5. Save

---

## 2. Gemini API (free tier)

1. Go to [aistudio.google.com](https://aistudio.google.com) → sign in
2. Click **"Get API key"** → **"Create API key"**
3. Copy the key — no billing required for the free tier
4. **Important**: use a Flash model (e.g. `gemini-2.5-flash`), not a Pro model — Pro models aren't included in the free tier and will return a `limit: 0` quota error
5. **Rate limits**: the free tier caps requests per minute. If testing rapidly triggers a "too many requests" error, wait a few minutes rather than retrying immediately

In n8n: **Credentials → Add → Google Gemini(PaLM) Api** → paste your key.

---

## 3. LinkedIn API

This is the most involved part — LinkedIn requires an app to be linked to a Company Page during registration, even for personal-use posting.

### 3.1 Create a placeholder Company Page (if you don't have one)
1. Go to `linkedin.com/company/setup/new/`
2. Choose **Company** → fill in any name, industry, and size
3. Create the page — this page can remain empty/unused, it exists purely to satisfy app registration

### 3.2 Create the LinkedIn App
1. Go to `linkedin.com/developers/apps` → **Create app**
2. Link the Company Page you just created
3. Upload a logo (required field)
4. Add a Privacy Policy URL (a GitHub profile URL works fine for personal projects)
5. Agree to terms → **Create app**

### 3.3 Add the "Share on LinkedIn" product
1. On your app's dashboard → **Products** tab
2. Find **"Share on LinkedIn"** → request access (auto-approves instantly)

### 3.4 Set up OAuth
1. Go to the **Auth** tab → copy **Client ID** and **Client Secret**
2. In n8n, create a **LinkedIn OAuth2 API** credential → paste in Client ID/Secret
3. n8n will display a **Redirect URL** (e.g. `https://oauth.n8n.cloud/oauth2/callback`) — copy it
4. Back in LinkedIn's Auth tab, paste that URL into **"Authorized redirect URLs"** → save
5. In n8n, click **Connect my account** → authorize via the popup

### 3.5 Configure the "Post to LinkedIn" node
- **Post As: Person** (not Organization — "Share on LinkedIn" Default Tier only grants permission to post as yourself; posting as an Organization requires a higher-tier product that needs LinkedIn's manual review)
- **Person**: select yourself from the auto-populated dropdown
- **Text**: `{{ $json.content.parts[0].text }}` (path depends on which AI node/response shape you're using — verify against your actual node's output)

---

## 4. Gmail (confirmation email)

1. In n8n: **Credentials → Add → Gmail** → OAuth2
2. Sign in with Google
3. **Watch the permissions screen carefully** — it must include "Send email on your behalf." If this is skipped or unchecked, you'll get a `403 - ACCESS_TOKEN_SCOPE_INSUFFICIENT` error later
4. If that error occurs after setup, go back to the credential and click **Reconnect**, paying attention to the scope this time

---

## 5. Import and Wire the Workflow

1. n8n → **Workflows → Import from File** → select `workflow/commitlog-workflow.json`
2. Attach each credential to its corresponding node
3. Confirm the chain: **Webhook → Generate LinkedIn Post → Post to LinkedIn → Send Confirmation Email**
4. Activate the workflow (top-right toggle)

---

## 6. Test It

1. Make a small, real commit to your tracked repo (use a realistic commit message — the AI summarizes whatever you give it, so "test 123" produces a useless test post)
2. Push it
3. Within seconds:
   - A post should appear on your LinkedIn profile
   - A confirmation email should land in your inbox

---

## Troubleshooting

| Issue | Likely Cause |
|---|---|
| Webhook never fires | Using Test URL instead of Production URL, or workflow isn't Active |
| GitHub sends data but n8n shows nothing | Content type set to `x-www-form-urlencoded` instead of `application/json` |
| Gemini returns `limit: 0` error | Using a Pro model instead of a free-tier Flash model |
| Gemini returns "too many requests" | Free-tier rate limit hit from rapid testing — wait a few minutes |
| LinkedIn node returns 403 Forbidden | "Share on LinkedIn" product not added, OR trying to post as Organization instead of Person |
| LinkedIn app creation blocked | No Company Page linked — this is a hard requirement, create a placeholder page |
| Gmail returns `ACCESS_TOKEN_SCOPE_INSUFFICIENT` | OAuth reconnect didn't grant the send-email scope — reconnect and check the permission screen explicitly |

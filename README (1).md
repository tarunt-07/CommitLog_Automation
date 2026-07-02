# 🔁 CommitLog — Turn Your Commits Into LinkedIn Posts

> A fully automated n8n workflow that watches a GitHub repo, has AI write a LinkedIn post about the update, publishes it live, and emails a confirmation — all triggered the moment you push code.

![n8n](https://img.shields.io/badge/n8n-automation-EA4B71?style=flat-square&logo=n8n&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-Webhooks-181717?style=flat-square&logo=github&logoColor=white)
![Gemini](https://img.shields.io/badge/AI-Gemini-4285F4?style=flat-square&logo=googlegemini&logoColor=white)
![LinkedIn](https://img.shields.io/badge/LinkedIn-API-0A66C2?style=flat-square&logo=linkedin&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)

---

## 🧠 The Problem

Every developer building side projects hits the same wall:

> You build something genuinely cool → you're excited for about 20 minutes → then you never actually post about it, because writing the update feels like a whole separate task.

**CommitLog** closes that gap. Push code, and an AI-written LinkedIn post goes live automatically — no copy-pasting, no "I'll post about it later" that never happens.

---

## ⚙️ How It Works

```
   git push
      │
      ▼
┌─────────────────────┐
│  GitHub Webhook      │  → fires in real time on every push
└──────┬───────────────┘
       │
       ▼
┌─────────────────────┐
│  Gemini (Google AI)  │  → writes a punchy, under-130-word LinkedIn post
└──────┬───────────────┘        based on the commit message
       │
       ▼
┌─────────────────────┐
│  LinkedIn API         │  → publishes the post live, automatically
└──────┬───────────────┘
       │
       ▼
┌─────────────────────┐
│  Gmail                │  → sends a confirmation email with the
└───────────────────────┘        commit details and post summary
```

1. **Push code** to the tracked repo
2. **GitHub webhook fires instantly** — no polling, no delay
3. **Gemini reads the commit message** and writes a short, sophisticated LinkedIn-style post — plain text, no markdown, casual-but-credible tone
4. **LinkedIn API publishes the post** live, automatically, as a Person post
5. **Gmail sends a confirmation** — commit message, repo name, timestamp — so there's always a record of what went out and when

---

## ✨ Why This Exists

I kept building things — a gesture-controlled AI drawing canvas, a contactless virtual keyboard, various coursework projects — and kept *not* posting about them because writing the update felt like its own task on top of the actual build.

This is my first end-to-end automation project. It's not about replacing the writing — it's about removing the friction between *finishing something* and *telling people about it*.

---

## 🛠️ Tech Stack

| Component | Role |
|---|---|
| [n8n](https://n8n.io) | Workflow orchestration |
| GitHub Webhooks | Real-time push trigger |
| [Gemini API](https://aistudio.google.com) (Google AI Studio) | AI post generation |
| LinkedIn API | Automated publishing |
| Gmail API | Confirmation delivery |

---

## 🚀 Setup

Full step-by-step setup guide is in [`docs/setup-guide.md`](docs/setup-guide.md) — including the LinkedIn Company Page requirement (yes, really — LinkedIn requires every app to be linked to a Page during registration, even for solo/individual use).

Quick version:

1. Clone this repo
2. Import [`workflow/commitlog-workflow.json`](workflow/commitlog-workflow.json) into your n8n instance
3. Connect your credentials:
   - GitHub webhook (repo → Settings → Webhooks)
   - Gemini API key (free tier via [aistudio.google.com](https://aistudio.google.com))
   - LinkedIn OAuth2 (requires a LinkedIn Developer App + "Share on LinkedIn" product)
   - Gmail OAuth2 (with `gmail.send` scope)
4. Activate the workflow
5. Push a commit and watch it flow through 🎉

---

## 📸 Live Example

*(Workflow canvas and a live generated post go here)*

---

## 🧗 The Hard Parts (a.k.a. what actually took the time)

Building the logic was the easy part. What actually ate the hours:

- **LinkedIn requires a Company Page just to register a developer app** — even for posting as an individual. Had to create a placeholder page to satisfy the requirement.
- **OAuth scope mismatches** — Gmail's default connection didn't include the `send` permission by default; had to reconnect with the right scope explicitly granted.
- **Gemini free-tier rate limits** — hit "too many requests" errors during rapid testing; the fix was simply pacing out test executions.
- **n8n expression paths** — every API (Gemini, LinkedIn, GitHub) returns data in a different JSON shape, so each node needed the exact right path (e.g. `$json.content.parts[0].text` for Gemini's response) or everything downstream broke silently.

If you're building something similar: budget real time for credential/permission setup — it's usually the actual bottleneck, not the automation logic itself.

---

## 🗺️ Roadmap

- [x] GitHub → Gemini → LinkedIn → Gmail pipeline (live)
- [ ] Add a Telegram/Slack approval step before auto-posting
- [ ] Filter out trivial commits (README-only changes, typo fixes)
- [ ] Weekly digest — roll up the week's commits into one post
- [ ] Support multiple repos with per-repo tagging

---

## 🤝 Contributing

This is an open-source, work-in-progress automation — PRs, issues, and suggestions are welcome.

---

## 📄 License

MIT — free to use, modify, and share.

---

## 👤 Author

Built by **Tarun** as a first automation project — part of an ongoing series of build-in-public side projects.

If this helped you, a ⭐ on the repo goes a long way.

# AI Telegram News Bot

An automated news pipeline built with **n8n**, **Google Gemini**, and the **Telegram Bot API**. The workflow fetches the latest news from an RSS feed, filters out duplicates, summarizes and classifies each article with AI, and delivers the results straight to a Telegram chat — fully hands-off, running on a schedule.

## Architecture

```
Schedule Trigger (every 1h)
        │
        ▼
     RSS Read  ──────────────►  Fetches latest items from an RSS feed (BBC World)
        │
        ▼
       Limit  ───────────────►  Caps the batch size to control API usage
        │
        ▼
 Remove Duplicates  ──────────►  Skips articles already processed in previous runs
        │
        ▼
  AI (Google Gemini)  ─────────►  Summarizes the article, classifies its topic,
        │                        and scores its importance (1–5)
        ▼
 Telegram (Send Message)  ────►  Delivers the formatted result to a Telegram chat
```

## Tech Stack

- **[n8n](https://n8n.io/)** — workflow automation / orchestration engine
- **Docker & Docker Compose** — local deployment of n8n
- **Google Gemini API** — AI summarization and classification
- **RSS** — news source
- **Telegram Bot API** — delivery channel

## How It Works

1. A **Schedule Trigger** runs the workflow every hour.
2. **RSS Read** pulls the latest articles from a news feed.
3. **Limit** restricts the batch to a manageable number of items to control API costs.
4. **Remove Duplicates** checks each article's link against previously processed items (using n8n's built-in execution state) so the same story is never sent twice.
5. New articles are passed to **Google Gemini**, which returns a structured JSON response with:
   - a short summary (2–3 sentences)
   - a topic category (e.g. Politics, Economy, Technology, World)
   - an importance score (1–5)
6. The formatted result is sent via the **Telegram Bot API** to a designated chat.

## Setup

### 1. Prerequisites

- Docker Desktop installed
- A Google AI Studio (Gemini) API key
- A Telegram bot created via [@BotFather](https://t.me/BotFather)

### 2. Run n8n locally

```bash
mkdir n8n && cd n8n
mkdir data
```

Create a `docker-compose.yml`:

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n
    ports:
      - "5678:5678"
    volumes:
      - ./data:/home/node/.n8n
    restart: unless-stopped
```

Start it:

```bash
docker compose up -d
```

n8n will be available at `http://localhost:5678`.

### 3. Import the workflow

Import [`workflow.json`](./workflow.json) into n8n via **Workflows → Import from File**.

### 4. Configure credentials

- **Google Gemini**: add your API key under Credentials.
- **Telegram**: add your bot's API token, and set the target `Chat ID` in the *Send a text message* node.

### 5. Activate

Set the Schedule Trigger interval to **every 1 hour**, then click **Publish** to activate the workflow.

## Project Files

| File | Description |
|---|---|
| `workflow.json` | Exported n8n workflow, ready to import |
| `docker-compose.yml` | Local n8n deployment configuration |
| `README.md` | This file |

## Roadmap / Possible Extensions

- [ ] PostgreSQL for persistent storage and history of processed articles
- [ ] Simple dashboard for reviewing sent news
- [ ] Multiple RSS sources with source-based routing
- [ ] Nginx + HTTPS for a public-facing deployment
- [ ] GitHub Actions CI to validate the workflow JSON on every commit

## Why This Project

Built as a portfolio piece to demonstrate practical automation and AI-integration skills: workflow orchestration, API integration, containerized deployment, and designing a pipeline that runs reliably and unattended.

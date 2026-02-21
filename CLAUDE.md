# Project: n8n Tutorial - Morning Briefing API

## Overview

Tutoriel pour decouvrir **n8n** (workflow automation) depuis zero sur Windows 11.
Le projet construit une API personnelle qui retourne un "briefing du matin" avec meteo + citation inspirante.

## Tech Stack

- **n8n** v2.8.3 (Community Edition) - launched via `npx n8n` on port 5678
- **Node.js** v18+ required
- No database, no Docker - pure npx local setup

## Project Structure

```
n8n-test/
  CLAUDE.md                    # This file
  README.md                    # Tutorial documentation (French)
  workflows/
    01-hello-world.json        # Simple workflow: Manual Trigger -> Edit Fields
    02-briefing-api.json       # Full briefing API workflow (6 nodes)
```

## Workflows

### 01-hello-world.json
- Manual Trigger -> Edit Fields (`{ "message": "Hello n8n!" }`)
- Import via n8n UI or CLI: `npx n8n import:workflow --input=workflows/01-hello-world.json`

### 02-briefing-api.json
- Webhook GET /briefing -> Get Quote (zenquotes.io) + Get Weather (Open-Meteo) -> Merge (by Position) -> Format Briefing (JS Code) -> Respond to Webhook
- Endpoint: `http://localhost:5678/webhook/briefing`
- Must be **published** (not just saved) for the webhook to be active

## External APIs

| API | URL | Auth | Notes |
|-----|-----|------|-------|
| ZenQuotes | `https://zenquotes.io/api/random` | None | Rate limit: 5 req/30s |
| Open-Meteo | `https://api.open-meteo.com/v1/forecast` | None | Worldwide coverage, WMO weather codes |

## Key Learnings & Gotchas

- **n8n Merge node v3**: When using "Combine" mode, always set "Combine By" to **"Position"** (not "Matching Fields") when merging parallel HTTP requests by their order. The JSON parameter is `"combinationMode": "mergeByPosition"` but the UI may default to "Matching Fields".
- **n8n CLI import**: `npx n8n import:workflow --input=<file>` always creates a NEW workflow (does not update existing ones). Edit existing workflows directly in the UI instead.
- **Webhook activation**: Workflows must be **published** (green dot) for production webhooks to respond. Draft mode only exposes test URLs.
- **n8n REST API auth**: Browser-based JS fetch to `/rest/` endpoints returns 401 even with `credentials: 'include'`. Use CLI commands or the UI instead.

## Commands

```bash
# Start n8n
npx n8n

# Import a workflow
npx n8n import:workflow --input=workflows/02-briefing-api.json

# Test the briefing API
curl -s http://localhost:5678/webhook/briefing

# Use a different port
npx n8n --port 5679
```

## Language

- README and code comments are in **French**
- Weather conditions in the API response are in French (via WMO code mapping)

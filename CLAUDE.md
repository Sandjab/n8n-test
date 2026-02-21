# Project: n8n Tutorial - Morning Briefing API

## Overview

Tutoriel pour decouvrir **n8n** (workflow automation) depuis zero sur Windows 11.
Le projet construit une API personnelle qui retourne un "briefing du matin" avec meteo, citation inspirante, actualites RSS et resume genere par IA.

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
    03-briefing-claude.json    # AI briefing: RSS + Claude integration (8 nodes)
```

## Workflows

### 01-hello-world.json
- Manual Trigger -> Edit Fields (`{ "message": "Hello n8n!" }`)
- Import via n8n UI or CLI: `npx n8n import:workflow --input=workflows/01-hello-world.json`

### 02-briefing-api.json
- Webhook GET /briefing -> Get Quote (zenquotes.io) + Get Weather (Open-Meteo) -> Merge (by Position) -> Format Briefing (JS Code) -> Respond to Webhook
- Endpoint: `http://localhost:5678/webhook/briefing`
- Must be **published** (not just saved) for the webhook to be active

### 03-briefing-claude.json
- Sequential flow: Webhook GET /briefing-ai -> Get Weather -> Get Quote -> Get News (RSS Le Monde) -> Build Prompt (Code) -> Call Claude (HTTP POST) -> Format Response (Code) -> Respond to Webhook
- Endpoint: `http://localhost:5678/webhook/briefing-ai`
- Requires Anthropic API key: replace `YOUR_ANTHROPIC_API_KEY` in the Call Claude node headers
- Uses `$('NodeName')` pattern to reference upstream node data (no Merge needed)
- Model: `claude-sonnet-4-20250514`

## External APIs

| API | URL | Auth | Notes |
|-----|-----|------|-------|
| ZenQuotes | `https://zenquotes.io/api/random` | None | Rate limit: 5 req/30s |
| Open-Meteo | `https://api.open-meteo.com/v1/forecast` | None | Worldwide coverage, WMO weather codes |
| Le Monde RSS | `https://www.lemonde.fr/rss/une.xml` | None | French news headlines |
| Anthropic (Claude) | `https://api.anthropic.com/v1/messages` | API key (`x-api-key` header) | Requires `anthropic-version: 2023-06-01` header |

## Key Learnings & Gotchas

- **n8n Merge node v3**: When using "Combine" mode, always set "Combine By" to **"Position"** (not "Matching Fields") when merging parallel HTTP requests by their order. The JSON parameter is `"combinationMode": "mergeByPosition"` but the UI may default to "Matching Fields".
- **n8n CLI import**: `npx n8n import:workflow --input=<file>` always creates a NEW workflow (does not update existing ones). Edit existing workflows directly in the UI instead.
- **Webhook activation**: Workflows must be **published** (green dot) for production webhooks to respond. Draft mode only exposes test URLs.
- **n8n REST API auth**: Browser-based JS fetch to `/rest/` endpoints returns 401 even with `credentials: 'include'`. Use CLI commands or the UI instead.
- **Sequential vs parallel**: Use sequential flow with `$('NodeName')` when sources have mismatched item counts (e.g. RSS returns N items). Avoids Merge complexity.
- **RSS Feed Read outputs multiple items**: The RSS node outputs one item per article. Place it last in a sequential chain to avoid multiplying downstream HTTP requests.
- **HTTP POST with JSON body in n8n**: Use `specifyBody: "json"` with `jsonBody: "={{ JSON.stringify($json.myObject) }}"` to send dynamic JSON bodies.

## Commands

```bash
# Start n8n
npx n8n

# Import a workflow
npx n8n import:workflow --input=workflows/03-briefing-claude.json

# Test the briefing API (step 2)
curl -s http://localhost:5678/webhook/briefing

# Test the AI briefing API (step 3)
curl -s http://localhost:5678/webhook/briefing-ai

# Use a different port
npx n8n --port 5679
```

## Language

- README and code comments are in **French**
- Weather conditions in the API response are in French (via WMO code mapping)
- Claude generates its briefing summary in French (prompt instructs French output)

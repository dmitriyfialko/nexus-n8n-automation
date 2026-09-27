# NEXUS — AI Content Automation with n8n

NEXUS is an automated editorial pipeline for a Russian-language Telegram channel about science, space and technology.

Instead of reposting RSS feeds, the system collects stories from multiple sources, scores their editorial value with an LLM, removes duplicate stories, generates original posts, prepares an image, schedules publication and publishes to Telegram.

## What this project demonstrates

- Multi-workflow orchestration in n8n
- PostgreSQL-backed processing state and queues
- LLM-based editorial scoring and structured output
- AI-assisted deduplication and post generation
- Hybrid image pipeline: source image first, FLUX fallback
- Retry/error branches for external APIs
- Time-zone-aware publishing schedule
- Telegram publishing with HTML formatting
- Central project settings provided by a Django service
- Separation of credentials from workflow logic

## Pipeline

```text
00 Orchestrator
   |
   +--> 01 Fetch RSS
   +--> 02 Score Articles
   +--> 03 Create Posts
   +--> 04 Generate Images
   +--> 06 Schedule Posts

05 Publish Posts runs independently on a short recurring schedule.
```

### Workflows

| Workflow | Responsibility |
|---|---|
| 00 | Runs the main pipeline on schedule and checks whether the project is enabled |
| 01 | Reads active RSS sources and inserts new articles into PostgreSQL |
| 02 | Scores fresh articles using an LLM and an editorial scoring model |
| 03 | Selects candidates, deduplicates stories and generates Telegram-ready posts |
| 04 | Uses a source image when possible and falls back to FLUX image generation |
| 05 | Publishes due posts to Telegram, records the message ID and cleans local files |
| 06 | Distributes READY posts across the daily publishing window |

## Stack

- n8n
- PostgreSQL
- Django
- DeepSeek
- Black Forest Labs FLUX
- Telegram Bot API
- Docker

## Repository structure

```text
workflows/   Sanitized n8n workflow exports
docs/        Architecture and setup notes
```

## Public workflow exports

The JSON files in this repository are portfolio-safe exports. Deployment-specific metadata and credential IDs are removed. Internal project IDs are replaced with placeholders and workflows are exported inactive.

After importing, reconnect credentials, configure the Django settings endpoint, and reconnect the sub-workflows used by the orchestrator.

## Status

This is a working project and an evolving portfolio case. The production deployment is intentionally not included in this public repository.

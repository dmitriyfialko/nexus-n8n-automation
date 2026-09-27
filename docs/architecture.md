# Architecture

NEXUS is split into small n8n workflows instead of one large workflow. PostgreSQL is the shared state layer between stages.

## Data flow

```text
RSS sources
   |
   v
articles (NEW)
   |
   v
LLM scoring
   |
   v
articles (SCORED)
   |
   v
candidate selection + deduplication
   |
   v
posts (DRAFT)
   |
   v
source image / FLUX fallback
   |
   v
posts (READY)
   |
   v
scheduler
   |
   v
Telegram publisher
   |
   v
posts (PUBLISHED)
```

## Control plane

Project-level settings are supplied by a Django internal API. n8n reads settings such as whether the project is enabled, the daily post limit, score threshold, publishing hours and API-backed configuration.

Secrets are stored outside workflow logic and referenced through n8n credentials or the trusted settings service.

## Editorial scoring

Articles are not filtered by one binary LLM decision. The scoring workflow produces an editorial profile with dimensions such as audience interest, novelty, impact, post potential and wow factor, plus an overall score.

## Image strategy

The image workflow is hybrid:

1. Prefer a relevant image from the source page when available.
2. Resize/re-encode source images before local storage.
3. If no usable source image is available, generate one with FLUX.
4. Poll the generation API with a retry limit.
5. Mark failures explicitly instead of silently dropping the post.

## Scheduling

The scheduler works in the Europe/Warsaw time zone, respects a configurable daily window and post limit, and stores scheduled timestamps as UTC instants.

## Security note

The public exports intentionally omit installation-specific IDs and secret values. They are examples of the production workflow design, not a ready-to-run production configuration.

# Wiki Log

> Chronological record of all wiki actions. Append-only.
> Format: `## [YYYY-MM-DD] action | subject`
> Actions: ingest, update, query, lint, create, archive, delete
> When this file exceeds 500 entries, rotate: rename to log-YYYY.md, start fresh.

## [2026-10-26] create | Wiki initialized
- Domain: AI Engineering Team
- Structure created with SCHEMA.md, index.md, log.md
- Tag taxonomy: architecture, development, models, qa, devops, tools, meta

## [2026-10-26] ingest | Android Code Review Rules
- Created entities/android-review-rules-bri.md (BRI-specific: MVI, BRITheme, Clean Architecture)
- Created templates/android-review-rules-template.md (generic Android review template)
- Source: BRI internal review guidelines document
- Tags: development, qa, android, standards

## [2026-10-27] ingest | PR Review Delegation Workflow
- Created entities/pr-review-delegation-workflow.md
- Documented: Bitbucket PR review via Hermes agent delegation + MCP fallback
- Process: delegate_task → MCP timeout → REST API fallback → review generation
- Tags: devops, tools, automation, workflow

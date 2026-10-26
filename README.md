# AI Team Backup

Auto-backup untuk Kanzan AI Team — Obsidian Vault, LLM Wiki, Hermes profiles.

## Struktur

```
.
├── obsidian-vault/       → symlink ke ~/Documents/Obsidian Vault
├── llm-wiki/             → LLM Wiki knowledge base
└── hermes-profiles/      → SOUL.md dari lead, dev, qa, devops
```

## Auto-Backup

Cron job jalan tiap hari 00:00 WIB (via DevOps Agent).

## Security

- Secrets (.env, tokens, API keys) **tidak** di-commit
- `config.yaml` tidak di-backup (ada token Telegram)
- Hanya SOUL.md + wiki content

## Team

- Lead Agent (Opus)
- Fullstack Dev (Gemini)
- QA Agent (Sonnet)
- DevOps Agent (Haiku)

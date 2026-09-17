# easypay-skill — public partner-facing AI skill

**⚠️ PUBLIC repository (MIT).** Everything committed here is visible to the world.

## What this is

The installable skill EasyPay partners give to their AI agent (Claude Code / Cursor /
Codex / Gemini CLI). It teaches the agent the EasyPay vocabulary and routes natural-language
requests to the 30 MCP tools served at `mcp.appload.tech`. The repo itself IS the product:
`SKILL.md` (agent instructions) + `README.md` (human install guide) + `DEPLOY.md`.

## Editing rules

- **Nothing internal leaks here**: no partner names, no API keys or key fragments, no
  internal hostnames/DB schemas/workflow names — only what is already public
  (`mcp.appload.tech`, `@easypay_onboarding_bot`, tool names).
- `SKILL.md` frontmatter `version:` bumps on every content change (semver-ish); mirror the
  version in the commit message.
- Tool list in `SKILL.md` must stay in sync with the live FastMCP toolset — when tools are
  added/renamed server-side, update here in the same change set.
- Russian example prompts are intentional (target audience) — keep the bilingual style.
- `git push` — только после явного «ок» владельца (публичный репозиторий).

## Skills

| Скилл | Когда использовать |
|-------|-------------------|
| `easypay-partners` | Выдать/проверить партнёрский API-ключ, capabilities — чтобы сверить toolset и инструкции |
| `easypay-db` | Проверить фактическое поведение тулов против прод-данных перед правкой описаний |

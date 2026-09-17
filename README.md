# EasyPay AI Skill

Connect your AI agent to **EasyPay** in one command. Speak natural language:

> «Создай новый продукт в Stripe «Продвинутый AI-курс» за $299 в месяц»
> «Создай для customer@example.ru ссылку на 25 000₽ через СБП»
> «Сколько у меня сейчас денег в долларах и крипте?»
> «Выставь банковский инвойс на $1200 john@example.com»
> «Организуй выплату подрядчику в РФ на 200 000 ₽ с наших долларов»

The skill teaches your agent the EasyPay vocabulary (Stripe / Mercury / Crypto / T-Bank, USD / EUR / GBP / BRL / RUB / CRYPTO, product lifecycle, payout flow) and which of the 30 MCP tools to call for each job.

---

## What you need

1. **An EasyPay partner account** with an API key.
   Don't have one? Open [@easypay_onboarding_bot](https://t.me/easypay_onboarding_bot) in Telegram → choose the **MCP** path.
2. **One of these CLI agents**: Claude Code, Cursor, Codex, or Gemini CLI.

---

## Install in 30 seconds

The API key goes into your MCP client config as an `X-Partner-Api-Key` HTTP header — **not into the chat**. The model never sees it; the FastMCP server at `mcp.appload.tech` validates and forwards it on every tool call.

Three CLI agents (**Claude Code / Codex / Gemini**) can install themselves — paste **one prompt**, the agent runs the commands. **Cursor** uses a JSON config paste because its MCP setup lives in Settings UI.

After install, the **first prompt** to send to your agent is universal:

```
Walk me through EasyPay onboarding so I can pick what fits my business and start accepting real payments.
```

For example prompts, alternative skill paths, and troubleshooting — see the [EasyPay MCP Customer Guide](https://docs.thenextgen.store/s/228aae3c-2e06-4b22-9982-8508a08d9d04).

### Claude Code

**1. The easy path** — paste this prompt into Claude and let it install everything:

```
Install the EasyPay MCP server and connect the skill for this project. To do this, run two commands — adapt them to your current environment if needed: claude mcp add easypay --transport http https://mcp.appload.tech/mcp/ --header "X-Partner-Api-Key: <YOUR_PARTNER_API_KEY>" ; mkdir -p .claude/skills/easypay && curl -fsSL https://raw.githubusercontent.com/EasyPay-Labs/easypay-skill/main/SKILL.md -o .claude/skills/easypay/SKILL.md
```

**2. Or install manually**:

```bash
claude mcp add easypay --transport http https://mcp.appload.tech/mcp/ --header "X-Partner-Api-Key: <YOUR_PARTNER_API_KEY>"
```

> If `claude mcp list` shows `easypay: ✗ Failed to connect`, run `claude mcp get easypay` — if it shows `Type: stdio` or `Command: \`, the shell mangled quoting. `claude mcp remove easypay` and rerun the exact command above (URL before `--header`, double quotes — works on bash and PS).

### Codex

Codex doesn't accept `--header` on `codex mcp add` — we bridge through the `mcp-remote` npm package (stdio ↔ HTTP+headers). Requires Node.js + `npx`.

**1. The easy path** — paste this prompt into Codex:

```
Install the EasyPay MCP server and connect the skill for this project. To do this, run two commands — adapt them to your current environment if needed: codex mcp add easypay -- npx -y mcp-remote https://mcp.appload.tech/mcp/ --header "X-Partner-Api-Key: <YOUR_PARTNER_API_KEY>" ; mkdir -p .agents/skills/easypay && curl -fsSL https://raw.githubusercontent.com/EasyPay-Labs/easypay-skill/main/SKILL.md -o .agents/skills/easypay/SKILL.md
```

**2. Or install manually**:

```bash
codex mcp add easypay -- npx -y mcp-remote https://mcp.appload.tech/mcp/ --header "X-Partner-Api-Key: <YOUR_PARTNER_API_KEY>"
```

### Gemini CLI

**1. The easy path** — paste this prompt into Gemini:

```
Install the EasyPay MCP server and connect the skill for this project. To do this, run two commands — adapt them to your current environment if needed: gemini mcp add easypay --transport http --header "X-Partner-Api-Key: <YOUR_PARTNER_API_KEY>" https://mcp.appload.tech/mcp/ ; mkdir -p .agents/skills/easypay && curl -fsSL https://raw.githubusercontent.com/EasyPay-Labs/easypay-skill/main/SKILL.md -o .agents/skills/easypay/SKILL.md
```

**2. Or install manually**:

```bash
gemini mcp add easypay --transport http --header "X-Partner-Api-Key: <YOUR_PARTNER_API_KEY>" https://mcp.appload.tech/mcp/
```

> If your Gemini version doesn't accept `--header` (rare — verified working 2026-05-10), use the `mcp-remote` bridge as in Codex.

### Cursor

Cursor's MCP install is a JSON config pasted into Settings → **MCP** → **Add new MCP server**:

```json
{
  "mcpServers": {
    "easypay": {
      "url": "https://mcp.appload.tech/mcp/",
      "transport": "http",
      "headers": {
        "X-Partner-Api-Key": "<YOUR_PARTNER_API_KEY>"
      }
    }
  }
}
```

The skill for Cursor goes into your project folder — see the [Customer Guide](https://docs.thenextgen.store/s/228aae3c-2e06-4b22-9982-8508a08d9d04) for the exact path.

---

## After install — full guide

Example prompts (Stripe products, crypto/bank invoices, payouts, balance), alternative skill paths (user-home vs project-local), per-CLI nuances, and troubleshooting all live in the [EasyPay MCP Customer Guide](https://docs.thenextgen.store/s/228aae3c-2e06-4b22-9982-8508a08d9d04) (RU + EN).

For products and invoices that need EasyPay moderation, you'll get a notification in your DM via `@easypay_onboarding_bot` once the link or invoice is live.

## Identity controls

Every call returns a successful HTTP response — the outcome is in the body. `{"success": true, ...}` means done; `{"success": false, "error_code": ..., "error_message": ...}` means it didn't happen. Three codes are worth knowing:

**`ACTOR_REQUIRED`** — money-moving operations (`create_partner_payout_request`) need an API key bound to a specific employee. A **personal** employee key submits payouts through the agent just fine; a **partner-wide** key doesn't. If you only have a partner-wide key, submit the payout from the EasyPay mini-app instead: [https://t.me/easypay_self_service_bot/dashboard](https://t.me/easypay_self_service_bot/dashboard). Read-only preview (`preview_partner_payout_options`) works with either key. `verify_partner_credentials` tells you which kind you're using (`auth_key_type`).

**`CROSS_TENANT_ATTEMPT`** — the `stripe_product_id` / `payment_link_id` / `recipient_id` your agent referenced exists but belongs to a different account (deliberately not a generic "not found"). Use `list_partner_invoiceable_products` / `list_partner_live_stripe_payment_links` to discover your own IDs.

**`DATA_INTEGRITY_ERROR`** — a server-side issue on EasyPay, not your mistake; the care team gets paged automatically, just retry after a moment.

Permission gating is enforced per endpoint — `verify_partner_credentials` returns your `permissions` array, so you can ask the agent "what's enabled for me?" instead of discovering it by trial and error.

---

## What's inside

- [`SKILL.md`](./SKILL.md) — the system prompt: 30 tools, EasyPay domain language, JTBD flows, anti-patterns.
- [`LICENSE`](./LICENSE) — MIT.

---

## Status

**Skill v0.9.0** (2026-07-24) — the skill is back in sync with the live server, in two ways.

*Catalog:* **26 tools**, matching `tools/list` exactly. Five were missing and are now documented — `create_partner_stripe_promotion_code`, `create_partner_mercury_invoiceable_product`, `create_partner_ruble_payable_product`, `list_partner_mercury_transactions`, `rotate_partner_webhook_secret` — along with flows for promo codes and webhook-secret rotation.

*Corrections:* payouts are **not** mini-app-only — a personal employee API key submits them through the agent (see "Identity controls"); `register_partner_notifications_webhook` registers your own HTTPS endpoints and never took a Telegram `chat_id`; and every tool answers with a successful HTTP response, so the agent must read `success` in the body rather than trust the status code. Backend openapi v0.7.0.

Transport: Streamable HTTP at `/mcp/` in stateless mode. The legacy SSE endpoint at `/sse` keeps working for existing installations but is no longer documented for new installs; switch to `/mcp/` at your convenience.

Install is agentic-primary (paste one prompt, the agent runs it) with a manual fallback per CLI. The skill is downloaded into your project (`.claude/skills/easypay/` for Claude Code, `.agents/skills/easypay/` for Codex & Gemini, project-local for Cursor — see Customer Guide). README is the install quickstart; the [Customer Guide](https://docs.thenextgen.store/s/228aae3c-2e06-4b22-9982-8508a08d9d04) is the source of truth for examples, alternative paths, and troubleshooting.

Not yet:
- CI markdown linter
- npm package
- Localizations beyond RU/EN

## Support

- Bot: [@easypay_onboarding_bot](https://t.me/easypay_onboarding_bot) — get an API key, ask for help with install.
- Direct: [@andre_erokhin](https://t.me/andre_erokhin)

## License

[MIT](./LICENSE) — use freely, including commercial. Attribution appreciated.

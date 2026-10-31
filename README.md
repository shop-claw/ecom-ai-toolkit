# Ecom AI Toolkit

Practical, copy-able guides for running a Shopify business with Claude: memory, ad and email connectors, store connection, seasonal prep and an audit skill.

| # | Guide | What you get |
|---|---|---|
| 1 | [Build a Store Brain](guides/01-store-brain.md) | A git-based memory for one store, the loop, the write gate |
| 2a | [Google Ads connector](guides/02a-google-ads-connector.md) | Official read server plus a safe write layer (stage, then confirm) |
| 2b | [Meta Ads connector](guides/02b-meta-ads-connector.md) | Official Meta Ads MCP, a DIY server, and a browser fallback |
| 3 | [Shopify connector](guides/03-shopify-connector.md) | How a Shopify store is wired to an AI operator, interactive and headless |
| 3b | [Klaviyo connector](guides/03b-klaviyo-connector.md) | Official Klaviyo MCP: remote and local setup, scopes, safe-send guardrails |
| 4 | [BFCM 2027 prep + prompts](guides/04-bfcm-2027-prep.md) | Data-driven plan from Shopify, Meta, Google Ads and Klaviyo data: pull, reconcile, strategy, staging, live monitoring |
| 5 | [CRO audit skill by ECORN](skills/ecorn-cro-audit/SKILL.md) | A full Shopify CRO audit procedure with GA4 and Clarity, as a Claude skill |

## Using the CRO audit skill

The skill is a single `SKILL.md`. Put the folder `skills/ecorn-cro-audit/` where your Claude setup loads skills (for Claude Code, a `.claude/skills/` folder in your project or your home directory), then ask for a CRO audit of a store. It expects browser automation for the live walkthrough and read access to the store's Shopify, GA4 and Clarity.

## Principles behind all of it

- Read first, write later. Every connector starts read-only.
- Preview, then confirm. Nothing that spends money or changes a live store happens without a human yes.
- Measure every change, and read the value back.
- Secrets never go in git or in chat.

## Contributing

Corrections and field notes are welcome. If an API or a menu path has changed, open an issue with the date and what you saw.

Licensed under MIT. Built by [ECORN](https://ecorn.agency).

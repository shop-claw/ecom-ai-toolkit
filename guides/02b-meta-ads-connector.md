# How to set up a Meta Ads connector for Claude

There are three routes. Start with the first one.

| Route | Effort | Best for |
|---|---|---|
| A. Meta's official Ads MCP server | Low | Most agencies and brands |
| B. Your own MCP server on the Marketing API | High | Hard guardrails, custom rules |
| C. Browser automation in Ads Manager | Medium | A fallback when A is not enabled for an account |

## Route A: the official Meta Ads MCP server

Meta runs its own MCP server for ads at `https://mcp.facebook.com/ads`. It was in open beta at the time of writing, so confirm current availability and pricing in Meta's documentation before you commit to it.

What it covers, per Meta's documentation: reporting, creating and managing campaigns, ad sets and ads, catalogs, pixels and datasets, help, A/B tests and lift studies, and the change log. The exact tool count varies by source, so read the list from the connector itself.

Behavior to know:

- New campaigns, ad sets and ads are created **paused**.
- Edits to existing objects apply **immediately**, and activating an object starts spend.

### Setup

1. In Claude, open the connector settings and add a **custom connector** with the URL above.
2. Sign in with Facebook and choose the business portfolios you want to expose.
3. In **Business Settings, the ads MCP server page**, review each ad account. You can switch action types on or off per account (the categories include budget, creation, targeting, creative and status). Turn off everything you do not want an AI to do. This is your main guardrail with the official route.
4. For agency work, the client must share their ad account with your business portfolio as a partner. Then confirm the account shows up in the ads MCP server page. Do not assume partner accounts are enabled; check each one.
5. Your own user must be assigned to the client ad account with the permission you need: view performance for analysis only, manage campaigns if you want Claude to create things.

### Gotchas

- Server-side System User tokens only work with the **Employee** role. Admin-role tokens are not accepted. If you wanted a headless service with an admin token, use Route B.
- The official route does not give you a mandatory confirm step, a budget cap or your own audit log. The per-account action switches are the substitute. Use them.
- If an account is listed twice in Business Settings (for example an old and a new one), check which one is the active one before you trust numbers from it.

### First-week protocol

1. Read-only summary on one account: spend, results, CPA, ROAS by campaign, last 7 days against the previous 7.
2. Compare the numbers with Ads Manager by eye.
3. Only then allow creation, on a test account, and review everything before activating.

## Route B: build your own MCP server

Build this only if you need rules Meta's server does not enforce. The design below is a reference, and a version built to it was tested against a mocked Graph API only, so treat it as a starting point.

### Tools (about 15)

- **Analysis:** list campaigns, ad sets, ads, pixels, audiences; `get_insights` for account, campaign, ad set or ad level, any date range, daily breakdown, and slices by country, age and placement. Compute purchases, CPA and ROAS on top of the standard metrics.
- **Creation:** campaign, ad set (targeting from simple inputs), creative with an image, ad; `search_targeting` to find interest, behavior and geo IDs.
- **Management:** `set_status`, `update_budget`.
- **Discovery:** `discover_ad_accounts` so you do not have to know IDs upfront.

### Guardrails (the reason to build it)

- Everything is created **PAUSED**.
- ACTIVE, ARCHIVED and any budget change require `confirm=true`.
- Before any write, verify the object belongs to the ad account of the client named in the call. One client can never be touched through another client's configuration.
- `MAX_DAILY_BUDGET_MINOR` as a hard cap. Budgets are in minor currency units (`2000` is 20.00 in USD, EUR or AED; zero-decimal currencies use whole units).
- `META_READ_ONLY=1` removes all write tools.
- `dry_run: true` on create tools returns the exact payload without sending it.
- Every write appended to an audit log: client, action, payload, result.
- HTTP mode refuses to start without a long random secret.

### Meta side, once

1. Create a Business-type app at developers.facebook.com and add the Marketing API.
2. Have each client share their ad account (and Page and pixel) with your business portfolio as a partner.
3. In Business Settings, Users, System Users: create a System User, assign the client ad accounts, and generate a token with `ads_read`, `ads_management` and `business_management`.
4. Meta may require Standard or Advanced Access for `ads_management` before it works on accounts outside your own business. Check the current Marketing API requirements. App Review can take time, so start early.

### Configure and run

```bash
cp .env.example .env              # MCP_SECRET, META_ACCESS_TOKEN
cp clients.example.json clients.json
npm install && npm run build
npm start                         # HTTP on :3000
npm run start:stdio               # for Claude Desktop, local
```

`clients.json` maps a short client key to its ad account, Page and pixel. Start empty and let `discover_ad_accounts` fill in the IDs.

### Deploy and connect

Any host that runs Node or a container works. Set the env vars there. In Claude, add a custom connector with `https://YOUR-HOST/mcp/<MCP_SECRET>`. The secret sits in the path because the connector form accepts only a URL. Use HTTPS only and rotate the secret if it leaks. For local use, register the server in `claude_desktop_config.json` with `--stdio`.

### Not covered in a first version

Image upload from a file (use a public URL or an existing image hash), video and carousel creatives, catalog and Advantage+ Shopping campaigns, creating custom audiences, scheduled reports. Meta also changes required fields between API versions, so pin the version and expect to fix the first real create call.

## Route C: browser automation

If the official connector is not enabled for a client account, Claude in Chrome can work in Ads Manager under your own login. It reads tables, switches accounts and walks the campaign wizard, and it can leave results as drafts or paused. It is slower and more fragile than an API, and a person must be signed in. Use it as a stopgap, and never let it press the final activate button without your yes.

## Example prompts

- "Compare the last 7 days with the previous 7 for this account, by campaign. Flag anything with ROAS under 2."
- "Which ads have the highest CPA this month? Break down by placement."
- "Build a prospecting campaign: sales objective, these countries, women 25 to 45, interest in specialty coffee, 30 USD a day. Dry run first, create paused."

## Permissions cheat sheet

| Goal | Minimum access |
|---|---|
| Reports only | View performance |
| Analyze and create | Manage campaigns |
| Never needed by an AI | Billing, and permissions of other people |

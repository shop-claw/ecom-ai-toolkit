# How to build a Google Ads connector for Claude

Goal: let Claude analyze a Google Ads account and also change it, without giving it a way to burn budget.

Google publishes an official, open-source Google Ads MCP server. It is read-only. The missing half is a write layer. The pattern below pairs the two.

```
            Claude / any MCP client
          reads  |             | writes (stage, then confirm)
   +-------------v--+     +----v------------------+
   | Official Google|     | Your write-layer MCP  |
   | Ads MCP (read) |     | (preview + confirm)   |
   +-------+--------+     +----+------------------+
           +--------+---------+
               Google Ads API
```

Use the official server for audits and reporting. Use the write layer to act on what you found.

A reference implementation of this design is `shop-claw/google-ads-powerful-mcp` (MIT). The guide below is written so you can build your own or adapt it.

## Design rules for the write layer

These rules matter more than any single tool.

| Rule | Why |
|---|---|
| **Stage, then confirm.** No write tool sends anything. Only `confirm_change` does. | The human sees the exact payload before money moves. |
| **Run `validate_only` at stage time.** | API errors appear before approval, not after. |
| **Staged changes expire** (for example after 15 minutes). | Stale approvals cannot be replayed. |
| **New campaigns are created PAUSED.** Enabling is a separate step. | Creation never spends. |
| **No destructive tools.** No remove or delete. Statuses limited to ENABLED and PAUSED. | A bad prompt cannot delete history. |
| **Hard budget cap** from an env var, applied to every budget created or changed. | Limits the blast radius of a typo. |
| **Geo targeting is mandatory** on new campaigns. | Nothing goes out to every country by accident. |
| **Account allowlist.** | Writes only reach accounts you listed. |
| **Read-only switch** (`READ_ONLY=true`). | Safe first week on a real account. |
| **Atomic create.** One `GoogleAdsService.Mutate` call per campaign. | If one part fails, nothing is half-created. |
| **Append-only audit log** of staged, confirmed, cancelled and failed changes. | You can always answer "who changed what". |

## Suggested tools

| Tool | Purpose |
|---|---|
| `create_search_campaign` | Stage budget, campaign (PAUSED), geo and language targeting, ad group, keywords, negatives, one responsive search ad |
| `add_keywords` | Stage keywords for an existing ad group |
| `add_negative_keywords` | Stage campaign-level negatives |
| `create_responsive_search_ad` | Stage an RSA in an existing ad group |
| `update_budget` | Stage a new daily amount (capped) |
| `update_campaign_status` | Stage ENABLED or PAUSED |
| `confirm_change` / `cancel_change` / `list_pending_changes` | The approval flow |
| `gaql_read` | Read-only GAQL SELECT, mainly to look up IDs before staging |

Keyword syntax in every tool: `[exact]`, `"phrase"`, or plain text using a default match type.

## Step-by-step setup

### 1. Google Ads API access

You need three things in your own Google accounts:

1. A Google Cloud project with the **Google Ads API** enabled.
2. An **OAuth client of type Desktop app** (APIs and Services, Credentials, Create credentials, OAuth client ID). If the consent screen is in Testing mode, add your Google account as a test user.
3. API access level for the project. Google has been changing how developer tokens and access levels work, so check the current Google Ads API documentation before you start. At the time this guide was written, the reference implementation relied on Explorer access (approved quickly, limited daily operations) and did not need a developer token. Verify this against Google's current docs, because it may have changed.

If you manage client accounts, work through a manager (MCC) account and make sure the accounts you will write to are linked under it.

### 2. Get a refresh token

Run the standard OAuth flow once with the `https://www.googleapis.com/auth/adwords` scope, signing in with the Google user that has access to the manager account. Store the refresh token in a file readable only by you, never in the Claude config.

### 3. Configure safety first

Before the server can do anything, set:

```
MAX_DAILY_BUDGET=50          # hard cap, in account currency
ALLOWED_CUSTOMERS=1234567890 # start with ONE test account
READ_ONLY=true               # flip to false only after a clean read test
```

### 4. Register both servers in Claude

Claude Code:

```bash
claude mcp add google-ads-write -- /abs/path/.venv/bin/python -m your_write_layer
claude mcp add google-ads-official -- pipx run --spec git+https://github.com/googleads/google-ads-mcp.git google-ads-mcp
```

Claude Desktop: add both under `mcpServers` in `claude_desktop_config.json`. Pass credentials through the `env` block of the server entry, not as plain text elsewhere.

### 5. Prove it in this order

1. Ask "what tools do you have?" and confirm both servers appear.
2. Run a read: top campaigns by spend, last 30 days.
3. Turn `READ_ONLY` off on the test account only.
4. Stage a change and read the preview and the `validate_only` result. Cancel it.
5. Stage again, confirm, and read the value back with `gaql_read`.

## Example prompts

- "In account 123-456-7890, find search terms with more than 50 spent and no conversions in the last 30 days, then stage them as negative keywords on the top three campaigns." (reads through the official server, writes through the write layer)
- "Create a Search campaign called 'Brand | US', 20 per day, geo 2840, language 1000, with these keywords, headlines and descriptions. Stage it, do not confirm."
- "Raise the budget of campaign 'Brand | US' to 30 and show me the preview."

## Looking up IDs

Geo and language IDs come from Google's constants. Common ones (verify before relying on them): United States 2840, United Kingdom 2826, Germany 2276, Spain 2724, UAE 2784; English 1000, German 1001, French 1002, Spanish 1003, Arabic 1019.

Any other location:

```
SELECT geo_target_constant.id, geo_target_constant.name, geo_target_constant.country_code
FROM geo_target_constant WHERE geo_target_constant.name = 'Spain'
```

## Known limits to state honestly

- Search campaigns only. Performance Max, Shopping, Demand Gen, asset groups and Customer Match need more work.
- Pending changes held in memory are lost on restart.
- Google changes API versions and required fields. Pin the API version.
- Test on a low-risk account first. Offline tests prove the operation builders, not your account's quirks.

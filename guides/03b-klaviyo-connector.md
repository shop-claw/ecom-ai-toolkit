# How to connect Klaviyo to Claude

Klaviyo runs an official MCP server. With it Claude can read your lists, segments, flows and campaign results, and can draft and change email and SMS work. This guide covers the two ways to connect, the permissions to grant, and the guardrails that keep an AI from sending something you did not approve.

Sources: Klaviyo's [MCP server overview](https://developers.klaviyo.com/en/docs/klaviyo_mcp_server) and [connection guide](https://developers.klaviyo.com/en/docs/connect_to_the_klaviyo_mcp_server). Klaviyo changes these pages, so check them for the current state.

## Pick a route

| Route | Auth | Best for |
|---|---|---|
| **A. Remote server (recommended)** | OAuth, no key to store | Most people. Works in the Claude app. |
| **B. Local server** | A private API key you create | Claude Desktop setups where you want to control the exact key scopes |

## Route A: the remote server

Server URL: `https://mcp.klaviyo.com/mcp` (no trailing slash). Transport is Streamable HTTP, authentication is OAuth. You need the **Owner, Admin or Manager** role in the Klaviyo account.

### One account: the listed connector

1. In Claude, open Settings, Connectors, Browse connectors.
2. Search for Klaviyo and connect.
3. Approve Claude's access, then review and allow the permissions for your Klaviyo account.

The listed connector does not accept query parameters, so you cannot set read-only mode this way.

### Several accounts or safety switches: a custom connector

This needs a paid Claude plan (Pro, Max, Team or Enterprise).

1. Settings, Connectors, Add custom connector.
2. Name it `Klaviyo`. For an agency, name it per client, for example `Klaviyo - Client A`.
3. URL: `https://mcp.klaviyo.com/mcp`. Add parameters to the URL, for example `?company=client-a`.
4. Add, then Connect, approve Claude, and allow the permissions in Klaviyo.
5. In each new chat, enable only the connector for the client you are working on.

### Query parameters worth knowing

| Parameter | Effect |
|---|---|
| `read-only=true` | Disables every tool that changes data |
| `disable-tools-with-user-generated-content=true` | Disables tools that read content written by customers |
| `core-tools-only=true` | Limits the server to roughly 30 core tools |
| `toolsets=profiles:read,campaigns:read` | Limits tools by API scope |

Combine them with `&`, for example `https://mcp.klaviyo.com/mcp?company=client-a&read-only=true`.

### Claude Code

Klaviyo's page does not cover Claude Code. The standard Claude Code command for a remote HTTP server is:

```bash
claude mcp add --transport http klaviyo "https://mcp.klaviyo.com/mcp?read-only=true"
```

Then authenticate from inside Claude Code when prompted. Confirm the syntax against the current Claude Code docs.

## Route B: the local server with an API key

Requires [uv](https://docs.astral.sh/uv/getting-started/installation/).

1. In Klaviyo, create a **private API key**. Give it only what you need. For all tools, Klaviyo lists these scopes:

| Scope | Access |
|---|---|
| Accounts | Read |
| Campaigns | Full |
| Catalogs | Read |
| Events | Full |
| Flows | Read |
| Images | Full |
| List | Read |
| Metrics | Read |
| Profiles | Full |
| Segments | Full |
| Subscriptions | Full |
| Tags | Read |
| Templates | Full |
| Translations | Full |

2. In Claude Desktop, open Settings, Developer, Edit Config and add:

```json
{
  "mcpServers": {
    "klaviyo": {
      "command": "uvx",
      "args": ["klaviyo-mcp-server@latest"],
      "env": {
        "PRIVATE_API_KEY": "YOUR_API_KEY",
        "READ_ONLY": "true",
        "ALLOW_USER_GENERATED_CONTENT": "false"
      }
    }
  }
}
```

3. Save, quit every Claude process, and relaunch.
4. Ask a harmless question to verify, for example "List my Klaviyo lists".

Notes:

- Start with `READ_ONLY` set to `true`. Flip it only when you have a reason.
- To avoid keeping the key in plain text, Klaviyo documents `uvx --from klaviyo-mcp-server@latest run-claude` (macOS and Windows) which asks for the key when it starts.
- If Claude Desktop will not start, use the full path to `uvx` in `command` (`which uvx` on macOS).
- Scope the key to the smallest set that works. A key for reporting needs no write scopes.

## Guardrails

The connector exposes more than reporting. Its tool list includes creating and editing campaigns, flows, templates and profiles, scheduling and sending campaigns, and bulk operations such as suppressing profiles and deleting objects. Treat that like any other write access to a live revenue channel.

1. **Start read-only.** Run a full week of audits and reports with `read-only=true` before you allow any write.
2. **Separate drafting from sending.** Let Claude create and edit drafts. A human opens the draft in Klaviyo, checks the audience, links and discount, and presses send or schedule.
3. **Never let an agent send unattended.** If you automate, write the rule down: agents design and stage, live sends need an owner's yes.
4. **Check the audience every time.** The most expensive mistake is the wrong segment. Ask Claude to show the list or segment name, the estimated recipient count and the suppression rules before you approve.
5. **Turn off user-generated content when you do not need it.** Customer-written text (reviews, form answers, profile properties) can contain instructions aimed at the AI. Use `disable-tools-with-user-generated-content=true` for routine work.
6. **One client, one connector, one chat.** Enable only the connector for the client you are working on.
7. **Use a dedicated Klaviyo user for the integration** where your plan allows it, so the activity log shows who did what.
8. **Rotate the API key** if it ever appears in a chat, a log or a ticket.

## First-week protocol

1. Read: list the flows and their status, then pull revenue per flow for the last 90 days.
2. Compare two numbers with the Klaviyo dashboard by eye. If they disagree, stop and find out why.
3. Ask for an audit: which core flows exist (welcome, abandoned cart, browse abandonment, post-purchase, winback), which are live, which have not been touched in a year.
4. Only then allow drafting, in a test list or a draft state.

## Example prompts

- "List all flows with status, trigger, and revenue over the last 90 days. Flag any core flow that is missing or in draft."
- "Compare the last 5 campaigns by revenue per recipient and click rate. What do the best two have in common?"
- "Draft a welcome email for the new-subscriber flow. Do not activate anything. Show me the subject, preview text and body."
- "Which segments are largest and when were they last used in a campaign?"
- "Before I approve this campaign: show the audience name, estimated recipient count, excluded segments and every link in the email."

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| Remote connection fails | Wrong URL (trailing slash) or your Klaviyo role is not Owner, Admin or Manager |
| Tools are missing | Config typo; quit all Claude processes and relaunch |
| Claude Desktop will not start | Use the full path to `uvx` |
| Context fills up fast | Too many tools enabled; use `core-tools-only=true` or `toolsets=` |
| Answers look wrong | Start a new chat and use specific names and date ranges |

## What the connector may not do

Some Klaviyo features are set up in the Klaviyo and Shopify interfaces, not through the API. For example, the Back in Stock flow depends on installing the Klaviyo and Shopify integration, adding Klaviyo's Back in Stock block to the product template, and turning the flow live. Check Klaviyo's current documentation for which steps are manual, and plan a human step for them.

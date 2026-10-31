# Connecting an AI operator to a Shopify store

This guide shows how to wire Claude (or any agent) to a Shopify store for both interactive and unattended work.

There are two lanes, and you will want both. They are not interchangeable.

| Lane | What it is | Writes | Use for |
|---|---|---|---|
| **Interactive** | The Shopify connector inside Claude (MCP) | Each write asks a human to confirm, by design | Exploring, audits, one-off fixes with you at the keyboard |
| **Headless** | A custom app with an Admin API token, called over HTTPS | Gated by your own autonomy rules | Scheduled, unattended work |

Reads in the interactive lane run without prompts. Writes there ask for a tap every time. That is a client behavior, not a config error you can fix, and it is why the unattended lane exists.

## Lane 1: the interactive connector

1. Add the Shopify connector in Claude's connector settings and sign in to the store.
2. Test with a read: store info, a product search, a small GraphQL query.
3. For a first write, pick something harmless and reversible, for example one product's SEO description, and confirm it by reading the value back.

Things that bite:

- Permission settings and hooks load at session start. Edit them, then restart the session.
- The connector can reconnect under a new dynamic ID, which breaks any rule pinned to the old name. Match tool names, not IDs.
- In some clients, "always allow" does not persist for connectors. Do not build a workflow that depends on it.

## Lane 2: the headless connector (custom app + Admin token)

### Create the app

1. In the Shopify admin, create a custom app for the store. Shopify has been moving app creation toward the Dev Dashboard, so the exact menu path may differ from older guides. Check Shopify's current documentation.
2. Grant the minimum scopes you need today. Start with:
   - `read_products`, `write_products` (titles, descriptions, tags, type, vendor, SEO, product metafields, image alt)
3. Add scopes only when a feature needs them:
   - `read_inventory`, `write_inventory`
   - `read_orders`
   - `read_content`, `write_content` (pages, blog)
   - `read_themes`, `write_themes` (structured data and `llms.txt` assets)
   - `write_publications` (publish and unpublish, keep this gated)
4. Install the app and reveal the **Admin API access token** once. It starts with `shpat_`. Copy that one.

Do not confuse these three values:

| Value | Looks like | Use |
|---|---|---|
| Admin API access token | `shpat_...` | **This is the one you need** |
| Client ID | a short ID | OAuth only |
| API secret key | `shpss_...` | OAuth only |

### Store the secrets

In the runner's secret store (GitHub Actions secrets, or your hosted routine's environment), never in git and never in chat:

```
SHOPIFY_ADMIN_TOKEN=shpat_...
SHOPIFY_SHOP=<your-handle>.myshopify.com
```

Use the real `myshopify.com` handle, not the public domain.

### Allow the network path

If your runner has an egress allowlist, add `*.myshopify.com`. One wildcard covers every store you will ever connect. Add the other hosts you use (Klaviyo, Google APIs, GitHub) in the same one-time setup.

### Declare it in the brain's config

```yaml
feeds:
  shopify:
    required: true
    access: admin_token        # the headless lane, not the interactive connector
    auth_env: SHOPIFY_ADMIN_TOKEN
    shop_env: SHOPIFY_SHOP
    writes: gated
```

Declaring every access in one manifest up front means you provision once, not one blocker at a time. A missing required access should fail the run immediately and loudly.

### Add telemetry before you open the write gate

The loop should not auto-apply changes it cannot measure. Connect these first:

- Search Console (organic clicks and impressions by page)
- GA4 (sessions, conversion), through a service account or OAuth refresh token stored as a secret

Then flip `require_telemetry_for_auto_apply` to true and let reversible changes through under a per-run cap.

## Prove the connection

Do these in order. Stop at the first failure.

1. **Read:** fetch shop info and ten products over the Admin API.
2. **Dry run:** produce a change-set for one product, apply nothing.
3. **Write one:** apply a single reversible change, such as an image alt text.
4. **Read back:** fetch the value again from Shopify and compare. A write is done only when the read-back matches.
5. **Revert:** re-fetch the live value, restore the old one, read back again.

## Rules that kept the store safe

- Reversible and measurable changes may auto-apply under a cap. Delete, archive and price changes always wait for a yes.
- Before any revert, re-fetch the live value. The store has no real undo.
- When a product flips between active and draft, re-verify the affected collection in the same run.
- Rotate any token that was ever pasted into a chat, a log or a ticket.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| 401 on every call | Used the API secret key or client ID instead of the `shpat_` token |
| 403 on a specific call | Missing scope; add it and reinstall the app |
| Works locally, fails in the runner | `*.myshopify.com` not in the egress allowlist |
| Writes ask for a tap in an interactive session | Expected; use the headless lane for unattended writes |
| Name-pinned permission rules stopped working | Connector reconnected under a new ID |

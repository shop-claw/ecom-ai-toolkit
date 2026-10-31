---
name: "ecorn-cro-audit-v2"
description: "Run an ECORN CRO audit of a Shopify store with GA4 and Clarity data, categorised and origin-tagged findings, luxury and low-volume modes, and a staged Stage 2 estimate. Use instead of ecorn-cro-audit."
---

# ECORN CRO audit (v2)

Produces a client-ready CRO audit of one Shopify store: an Artifact page plus an A4 PDF in ECORN branding, backed by a live browser walkthrough, Shopify analytics, GA4 and Microsoft Clarity behaviour data including session recordings. Never ship a finding that was not observed directly.

This is Stage 1 of a staged engagement: (1) audit and prioritised recommendations, (2) implementation of approved items at the hourly rate, (3) measurement, validation and refinements. The audit must hand the client a list they can approve item by item, and a baseline that Stage 3 measures against.

Client-facing copy is in English, short and direct, no em dashes.

## 0. Intake (ask once, then proceed)

From the sales thread and the discovery call notes (Granola, Fathom), extract before starting:

- Store URL, store name, who the report is for, agreed price and delivery date.
- What the client explicitly asked to be included in the report. Treat every such request as a required section and quote it back in the summary.
- Site history: launch date, base theme, who customised it (agency, freelancer, in-house), changes since launch.
- Business shape: AOV and price range, orders per month, traffic mix (paid vs organic), markets and currencies.
- Tools the client says they use, and tools they say they do not use (for example "we do not do email"). These get checked against the installed app list in section 1.
- Whether a speed audit exists or is planned (the two reports cross-reference each other).
- Hour budget for the roadmap. Default 80 to 100 h total including speed work.

Set two modes from this, and write them at the top of the working notes:

- **Positioning mode:** `standard` or `luxury`. Luxury when AOV is high (roughly $250+) or the brand sells on scarcity, craft or status. Luxury mode bans discounts, countdown timers, fake scarcity and coupon popups from every recommendation.
- **Volume mode:** `testable` (a few hundred orders a month or more) or `low-volume`. Low-volume changes the testing plan in section 10.

## 1. Access request and verification

### 1.1 What to request (read-only everywhere)

- **Shopify:** collaborator access. Ask for the myshopify.com URL and the collaborator request code (Settings, Users and permissions, Security). Send the request from the Partner Dashboard with view scopes: Analytics and Reports, Orders, Products, Customers, Themes (including code), Apps, Settings.
- **GA4:** Viewer or Analyst on the property.
- **Microsoft Clarity:** team invite (Settings, Team), Viewer or Member.
- **Klaviyo or other ESP**, if installed: Analyst or read-only.
- **Ad accounts**, if paid traffic exists: Analyst or read-only. **GTM:** Read. **Search Console:** Restricted user.
- Never accept passwords. Invites to a dedicated audit mailbox only, never a personal account.

State in the request that the delivery clock starts when all access is verified.

### 1.2 Verify before starting the clock

Use Claude in Chrome in the Chrome profile where the audit mailbox is signed in. If a tool shows a sign-in page, the session is in the wrong profile: ask the person to sign in there, do not work around it.

Google products open on whichever Google account is first in the browser. Open the account switcher (avatar, top right), choose the audit mailbox, and note the `authuser=N` value in the resulting URL; use that URL for the rest of the audit. If the client property is missing from the picker, check the signed-in account before concluding access is missing.

Check and record:

- Shopify: Home opens on the store, Analytics, Reports, Online Store themes with Edit theme, Settings, Apps all load.
- GA4: the client property opens, Reports and Explore load, role visible from what Admin shows. Open Admin, Data retention and record the setting (see section 3).
- Clarity: project dashboard, Recordings, Heatmaps load; note the role shown under Settings, Team.

Then, from the installed app list, compare against what the client said. An installed ESP when the client says "no email" is either a finding (no recovery flows) or a hidden setup: ask, and request read access.

Also open Shopify Analytics, Reports and note any reports created by the client. Read those first: they show what the client is worried about.

## 2. Shopify data (source of truth for orders and conversion)

Path A, connector: if the Shopify connector is live for this store, pull with `run-analytics-query` (ShopifyQL) and `graphql_query`. If it needs to sign in again, say so and ask the person to reconnect it.

Path B, admin in Chrome or exports. If neither works, send this export request first and start the walkthrough while waiting:

> In Shopify, Analytics, Reports, last 90 days compared with the previous period:
> 1. Conversion rate breakdown (sessions, added to cart, reached checkout, completed checkout)
> 2. Sessions over time and Conversion rate over time, monthly
> 3. Conversion rate by landing page type
> 4. Sessions by landing page, and Bounces by landing page URL, last 30 days
> 5. Sessions by device type, Sessions by referrer, Total sales by referrer
> 6. Average order value over time, and returning customer rate

Pull: sessions, added to cart, reached checkout, completed checkout, conversion rate, AOV, returning customer rate, by month for 90 to 100 days; conversion by landing page type; sessions by landing page URL; sessions and sales by referrer, device and country; bounces by URL including UTM variants.

Read for these patterns before writing:

1. Conversion trend inside the period, not only year on year; flag seasonality.
2. Where each traffic source lands and how that landing page converts against the site average.
3. Which funnel step leaks: session to cart, cart to checkout, checkout to order.
4. Channels and countries with traffic and no sales.
5. Recovery flows showing $0.
6. Bot-shaped traffic: data-center cities (Council Bluffs, Ashburn, The Dalles), direct spikes with 100% bounce. Cross-check with GA4 cities and Clarity bot share.
7. Mobile share, which sets the priority of every fix.
8. Content and blog traffic with no path to product (curiosity searches landing on articles).

## 3. GA4 data (attribution, engagement, tracking health)

GA4 answers what Shopify reports answer roughly: which source and campaign brought each session, how engaged it was, and which events fired. Shopify stays the source of truth for orders and revenue.

### 3.1 Setup checks first (they decide how far the data can be trusted)

- **Data retention** (Admin, Data retention). If it is 2 months, Explorations cannot reach a 90-day window: take the 90-day view from standard reports and Shopify, and use Explore only inside the retention window. Note the setting in the report and recommend 14 months.
- **Shopify integration and tracking changes.** Read the Home banners, Admin, Data streams and Events for the date the Shopify integration or a new tag was enabled. Do not compare periods across a tracking change; split the window at that date.
- **Duplicate or missing purchase tracking.** Compare GA4 purchases and purchase revenue with Shopify orders and net sales for the same dates. Within roughly 15% is normal (consent, ad blockers). Far below: tracking loss. Above Shopify: duplicate purchase events (theme tag plus app plus integration).
- **Key events.** List which events are marked as key events; if none or only page views, say so.
- **Data filters and internal traffic, referral exclusions, cross-domain** (checkout and payment providers showing up as referrers means self-referrals are breaking attribution).
- **Consent settings**: consent mode on or off, and whether a consent banner is live on the storefront.

### 3.2 What to pull

Use the same 90 days as Shopify (or the retention window), split at any tracking change.

- Acquisition, Traffic acquisition: sessions, engaged sessions, engagement rate, average engagement time, purchases, revenue by session default channel group and by session source / medium.
- Engagement, Landing page: sessions, engagement rate, purchases per landing page; then add session source / medium as a secondary dimension for the top 10 landing pages.
- Demographics: country and city with engagement rate; flag data-center cities and countries with high sessions and zero engagement.
- Tech: device category, browser, OS with engagement rate and purchases.
- Explore, Funnel exploration: session_start or page_view, view_item, add_to_cart, begin_checkout, add_shipping_info, add_payment_info, purchase. Open funnel, breakdown by device category, then a second copy broken down by session default channel group. If ecommerce events are missing, the funnel itself is a tracking finding.
- Explore, Path exploration from the top 3 landing pages (organic and blog entry in particular): what the next pages are, and how many sessions end on the landing page.
- Site search: `view_search_results` with `search_term`, if collected.
- AI and assistant referrers: chatgpt.com, perplexity.ai, copilot, gemini as sources, sessions and engagement.
- Search Console link (Admin, Product links): if linked, organic queries by landing page.

Export each table to CSV or copy it with `get_page_text`, save it to the working folder, and compute in Python.

### 3.3 How GA4 feeds the report

- Traffic quality category: engagement rate and purchases by source and landing page, bot-shaped cities and countries, self-referrals.
- Touchpoint map: entry page, next step and drop point per source come from landing page plus path exploration.
- Findings: tracking gaps (missing ecommerce events, duplicate purchases, no key events, short retention, broken attribution) are Technical defects with origin Config or App or script.
- Funnel decomposition: Shopify gives the counts, GA4 gives the split by device and channel. When the two disagree, report Shopify and explain the gap.

## 4. Clarity data (behaviour, bots, errors, recordings)

Open the project dashboard with `?date=Last%2030%20days` and read it with `get_page_text`. Record:

| Block | What to take |
|---|---|
| Traffic | Sessions, bot sessions excluded, bot share, bot types (suspicious interaction, suspicious device, scrapers, suspicious origin) |
| Engagement | Pages per session, scroll depth, active time vs total time |
| Users | New vs returning share, unique users, top countries |
| Frustration | Rage clicks, dead clicks, quick backs, excessive scrolling, each with session counts |
| Referrers | Source list, including AI referrers |
| Devices | Browser, device, OS, region split |
| Real-user Core Web Vitals | Performance score, LCP, INP, CLS and the share of good / needs improvement / poor pageviews |
| JavaScript errors | Total, error share of sessions, top messages with their share |
| Pages | Top pages, top entry pages, popular products |
| Smart events and funnels | Counts only, see rules below |

Then qualitative work:

- **Recordings:** watch 10 to 15 sessions filtered by quick back, dead click, rage click, mobile plus product page, sessions with JS errors, and sessions from the top organic entry page. For each, note the timestamp, what the visitor tried to do and where they stopped. Screenshot the key frame with `save_to_disk` and use it as evidence in the finding card.
- **Heatmaps:** home, top collection, top two product pages and the top blog entry page, on mobile and desktop. Screenshot click and scroll maps with `save_to_disk`.

Rules for Clarity data:

- Conversion is calculated from Shopify only. Clarity smart events and funnels often undercount. Say this once as a measurement note, not as a finding.
- Bot share goes into the Traffic quality category and is compared with Shopify sessions and GA4 cities; if bots are not filtered in Shopify, real conversion is higher than reported and the report says so.
- Map each top JS error to the script host or app it comes from (section 5.3).
- Put Clarity real-user CWV next to the Shopify admin web performance panel (top of the Online Store themes page). If they disagree, explain the difference in sample and device mix.
- Data export API: Settings, Data export, 10 calls per project per day, last 1 to 3 days only. Do not generate a token in the client project without asking: it is a standing setting in their account.
- Clarity keeps recordings and dashboard data for about 30 days, so 90-day trends come from Shopify and GA4.

## 5. Live walkthrough (always)

Use a browser automation tool (Claude in Chrome in the original setup). Load the tools in one call: `tabs_context_mcp, navigate, computer, get_page_text, javascript_tool, browser_batch, read_console_messages, find, tabs_close_mcp`. A `browser_batch` cannot act on a page after navigating to it: navigate alone, then batch the actions.

Walk: home, top two product pages, collection pages, every nav item, search, cart drawer, the top blog articles by traffic, contact and policy pages, and the top entry page for each traffic source. Capture each with `computer` and `save_to_disk: true`.

### 5.1 Collect with `javascript_tool`

Keep output free of query strings, or the extension blocks the result.

- theme and platform: `window.Shopify.theme` (name, id, theme_store_id, role), `<meta name=generator>`
- catalog: `fetch('/products.json?limit=250')` and `fetch('/products/<handle>.js')` for variant prices, compare-at, stock and option counts
- headings, nav links with hrefs, forms, image count, missing alt, image natural sizes on product pages, JSON-LD types and whether `aggregateRating` exists
- performance: `performance.getEntriesByType('navigation')`, resource count, transferred KB, third-party hosts
- tracking on the page: which GA4 measurement IDs and tag managers load (count of `gtag` configs, GTM containers), to back up any duplicate-tracking finding from section 3.1
- review widget presence (badge only vs full widget)
- console errors: `read_console_messages` with `onlyErrors: true` on every template
- add to cart: click it, `fetch('/cart.js')` to confirm, then `fetch('/cart/clear.js', {method:'POST'})`. Always clear the test cart.

### 5.2 Checkout and region simulation

Goal: catch price, currency, shipping and availability gaps by country without placing an order or entering personal data.

- Admin: Markets (countries, currencies, price adjustments), Settings, Shipping and delivery (zones, rates, free shipping thresholds), Settings, Checkout.
- Storefront: switch country through the store's localization selector, then re-read `/products/<handle>.js` and `/cart.js` for currency and price per market.
- With an item in the test cart, request shipping rates for the top countries from Shopify and GA4 (city or postal code level only), for example via `/cart/prepare_shipping_rates.json` then `/cart/async_shipping_rates.json`.
- Go to checkout only to see the first step and its trust elements. Never type an address, email or payment data, and never submit. Clear the cart afterwards.
- Record: countries where shipping is unavailable or very expensive relative to AOV, currency mismatches, duties or tax surprises, and how this matches sales and engagement by country.

### 5.3 Origin of technical problems

For every technical finding, assign an origin and a confidence:

- **Base theme:** behaviour present in the stock version of the same theme and version.
- **Agency or custom code:** files, sections or snippets not in the stock theme, or edits to stock files.
- **App or third-party script:** comes from an app embed, app block, or a script host owned by an app; also leftovers from uninstalled apps.
- **Store configuration or content:** settings, markets, shipping, tracking setup, missing images, copy.

Method: from `window.Shopify.theme` get the theme and whether it came from the Theme Store; in Edit code compare file list and modified files against a fresh copy of the same theme; map console and Clarity JS errors to their script host; list app embeds in the theme editor; search theme code for asset URLs of apps no longer installed and for hard-coded tracking tags.

Confidence: **confirmed** when seen in code or reproduced; **likely** when inferred from symptoms. Without code access, everything is likely, and the report says so.

### 5.4 Mobile

`resize_window` usually fails on a maximized window. On a same-origin page, replace the document with 390px iframes whose `srcdoc` is the fetched HTML plus a `<base href>`; X-Frame-Options does not apply to srcdoc. Screenshot the row of frames. Measure sticky bars and overlapping widgets there. Back each mobile finding with a Clarity mobile recording or heatmap where one exists.

## 6. Touchpoint map and competitor offer analysis

**Touchpoint map:** one table per traffic source (organic search, direct, social, AI referrers, email, paid if any): typical entry page (GA4 landing page by source), what the visitor sees first, next step offered, where they drop (GA4 path exploration, Shopify landing page conversion, Clarity quick backs and recordings), and the missing step. Include blog and content pages.

**Competitor offer analysis:** 3 to 5 direct competitors or category leaders. For each: entry price and smallest size, hero product price, sampling or gifting option, shipping and returns promise, guarantee, proof (press, reviews, origin story), content and education, email capture offer. Show it as one comparison table and end with the 2 or 3 gaps that matter for this store.

## 7. Findings

One card per finding, ordered by revenue impact:

- severity pill: Critical, High, Medium
- category chip: **Technical / usability defect**, **Conversion opportunity**, or **Traffic quality**
- for technical findings, an origin chip (Base theme, Custom code, App or script, Config or content) and Confirmed or Likely
- headline that states the defect
- screenshot, heatmap or recording frame with a caption naming what to look at
- What / Evidence (Shopify number, GA4 number, Clarity number or recording) / Cause / Impact / Fix with hour estimate

Critical means it blocks or misdirects revenue today: blank or broken templates, dead nav, a traffic source landing on a page that cannot convert, checkout or shipping failures in a selling market, purchase tracking so broken that decisions cannot be measured.

Recurring checks every time: hidden variant price tiers; review count with no widget and no rating schema; proof and guarantees below the fold; product images under 600px on a premium product; popups with no offer; cart with no add-on and no shipping progress; stale seasonal copy; sold-out items in the menu; third-party launchers covering buttons on mobile; JS errors on product or cart templates; blog articles with no product path; installed apps the client does not use; GA4 purchases not matching Shopify orders; short GA4 retention.

In the executive summary, show counts per category and the top three items per category, because the client may have asked for exactly this split.

## 8. Growth section

### Standard mode

- **Tripwires** for cold traffic: a consumable bundle, a kit, or a digital product, priced 5 to 15% of the hero product, credited toward the hero product. Say where each is deployed: retargeting, popup replacement, cart, post-purchase.
- **Quiz:** 5 questions mapping to 3 recommendations, each with a follow-up email flow. Entry points: menu, homepage block under the hero, popup, ads landing page. Build with an app template.
- **Email:** quiz result, tripwire to hero product, cart and checkout recovery.

### Luxury mode

No discounts, countdowns, fake scarcity or coupon popups. Instead:

- **Entry product and gifting:** the smallest size or a tasting set as the first purchase, gift packaging and gift messaging, a gifting guide.
- **Editorial and storytelling:** origin, process, people, rarity explained with high-end visual context; lifestyle imagery; customer stories and UGC; press and brand associations as trust.
- **Content to commerce:** on high-traffic curiosity articles, embed a product highlight, a short quiz, and an email or community capture.
- **Guided selling:** a quiz or AI-assisted recommendation that ends in one product, not a list.
- **Longer conversion window:** email or community capture with a reason to join (story series, release notes, allocation access), and retargeting of engaged visitors on the 2nd and 3rd touchpoints.
- **Boutique service:** concierge contact, white-glove follow-up on every order, LTV over first-visit conversion.

## 9. Impact model

Use the store's own average month (sessions excluding bots where known, conversion rate, AOV). Compute conservative / expected / upside from per-lever ranges in Python, never by hand. State ranges as benchmark estimates, not promises. Add one line on what is not counted (repeat purchases, recovered checkouts). In low-volume mode, also show the effect of a single extra order per month, because one order can move the conversion rate visibly.

## 10. Roadmap, Stage 2 estimate and testing

Phases 0 to 5: hotfixes and tracking fixes (settings only, first week), speed work, product page, growth features, email, then validation. Show a 12-week bar chart plus a table with scope and weeks, a "later (optional)" row, and a total line splitting CRO hours from speed hours.

Every roadmap item becomes a line in the **Stage 2 estimate**: ID, finding it fixes, category, hours, cost at the agreed rate, dependency. The client approves lines, not phases. Include a not-to-exceed total.

Phase 0 always includes: fix tracking gaps found in section 3.1 so Stage 3 can be measured, and connect the theme to GitHub so every change is versioned and reviewable; the client talks to the strategist and PM, the developer works through the repo.

Prefer theme sections, app settings and app templates over custom code, and say so in the scope column.

**Testing plan by volume mode:**

- Testable: 6 hypotheses with primary metric and run weeks. Judge on add-to-cart rate, click-through and revenue per visitor, two weeks minimum. Tools: Shoplift or Intelligems; Meta experiments for ad-side tests.
- Low-volume: no split tests, say why in one sentence. Use sequential rollouts with a recorded baseline, micro-conversions tracked in GA4 (product view to add-to-cart, quiz completion, email capture, scroll to proof block), Clarity recordings before and after, and a per-order review.

**Stage 3 baseline:** a table of the metrics to re-measure (sessions excluding bots, conversion rate, add-to-cart rate, AOV, engagement rate by channel, email capture rate, quick backs, JS error share, INP), their current value, the source (Shopify, GA4 or Clarity), the date range, and when to re-check.

## 11. Build the document

The design system ships inside this skill (sections 11.1 to 11.3).

### Working folder

```
work/
  src/report.html      body fragment with {{CSS}}, {{logo}}, {{img:name}} placeholders
  src/brand.css        section 11.1
  img/ecorn-logo.png   white ECORN wordmark
  img/<name>.jpg       screenshots, 1200px wide, quality ~74
  data/                Shopify, GA4 and Clarity tables as CSV
  fonts/               npm pack @fontsource/roboto-condensed @fontsource/jetbrains-mono
  build.py             section 11.2
  render.py            section 11.3
  out/                 built html + pdf
```

Logo: from ecorn.agency in Chrome (`img[alt="ECORN logo"]`, zoom screenshot with `save_to_disk`) or reuse a previous audit folder.

Structure of `src/report.html`: `<title>`, `<style>{{CSS}}</style>`, then `header.cover` (brandbar with logo and tag, eyebrow, h1, sub, meta grid), `main.page` with numbered `section` blocks, a `.printsign` line just before `</main>`, then `footer.closing`.

Suggested section order: 1 Summary (with category counts and what the client asked for), 2 Numbers (Shopify funnel, GA4 by channel and device, Clarity behaviour, bot share, tracking health), 3 Touchpoint map, 4 Findings by category, 5 Origin of technical problems, 6 Regions, pricing and shipping, 7 Competitor offers, 8 Growth plan, 9 Impact model, 10 Roadmap and Stage 2 estimate, 11 Testing and Stage 3 baseline, 12 What was not tested.

Classes available: `.summary`, `.stats/.stat` (`.bad`, `.good`), `.finding` with `.pill.crit/.high/.med/.ok`, `.chip` and `.chip.cat` / `.chip.origin` for category and origin tags, `dl.row` with `dt.fix`, `figure > .shot` (`.bar` + `.dots` + `.url` + img, or `.shot.blank` with `.empty`), `.pair`, `.grid2/.card`, `.tablewrap/table` with `td.num/.bad/.good` and `tr.total`, `.hbars/.hb` and `.funnel/.fr`, `.flow/.step`, `.gantt/.weeks/.track`, `.callout` (`.green`), `.note`, `.pagebreak`.

Rules: uppercase cover headline; no pixel font; no em dashes; screenshots always inside a browser frame with the URL; a page that renders blank shown as an empty captioned frame; one committed look, no dark mode.

### 11.1 brand.css

```css
:root{
  --black:#090909; --black-2:#151515; --black-3:#232323;
  --paper:#f3f3f1; --card:#ffffff; --ink:#0d0d0d; --ink-2:#4a4a4a; --line:#dedede;
  --violet:#c48ef6; --violet-ink:#7a3fd0; --violet-soft:#f1e8fd;
  --green:#06b178; --green-ink:#047a53; --green-soft:#e2f6ee;
  --crit:#c9302c; --crit-soft:#fbe6e5; --high:#b8650a; --high-soft:#fcefdd; --med:#5a4a86; --med-soft:#ecebf5;
  --display:"Roboto Condensed","Arial Narrow",Arial,sans-serif;
  --sans:"Roboto Condensed","Arial Narrow",Arial,sans-serif;
  --mono:"JetBrains Mono",ui-monospace,Menlo,monospace;
}
*{box-sizing:border-box}
html{-webkit-print-color-adjust:exact;print-color-adjust:exact}
body{margin:0;background:var(--paper);color:var(--ink);font:400 17px/1.55 var(--sans)}
.page{max-width:920px;margin:0 auto;padding-inline:20px}
h1,h2,h3{margin:0;text-wrap:balance;line-height:1.15}
p{margin:0}
a{color:var(--violet-ink)}
code{font-family:var(--mono);font-size:.82em;background:var(--violet-soft);color:#3b1d6b;padding:1px 5px;border-radius:3px;word-break:break-word}
.cover{background:var(--black);color:#fff;padding-block:28px 44px}
.cover .page{display:flex;flex-direction:column;gap:28px}
.brandbar{display:flex;justify-content:space-between;align-items:center;gap:16px;flex-wrap:wrap;border-bottom:1px solid var(--black-3);padding-bottom:18px}
.brandbar img{height:46px;width:auto;display:block}
.tag{font:500 11px var(--mono);letter-spacing:.14em;text-transform:uppercase;color:#9a9a9a}
.cover .eyebrow{font:500 12px var(--mono);letter-spacing:.16em;text-transform:uppercase;color:var(--green)}
.cover h1{font:700 clamp(36px,6.4vw,62px)/1.02 var(--display);color:var(--violet);letter-spacing:-.01em;text-transform:uppercase}
.cover .sub{font-size:20px;color:#d8d8d8;max-width:62ch}
.meta{display:grid;grid-template-columns:repeat(auto-fit,minmax(160px,1fr));gap:1px;background:var(--black-3);border:1px solid var(--black-3);border-radius:10px;overflow:hidden}
.meta div{background:var(--black-2);padding:12px 14px;display:flex;flex-direction:column;gap:2px}
.meta span{font:500 10.5px var(--mono);letter-spacing:.12em;text-transform:uppercase;color:#8a8a8a}
.meta b{font-weight:600;color:#fff}
main.page{display:flex;flex-direction:column;gap:48px;padding-block:44px 64px}
section{display:flex;flex-direction:column;gap:18px}
.sechead{display:flex;flex-direction:column;gap:6px;border-bottom:2px solid var(--ink);padding-bottom:10px}
.sechead .n{font:500 11px var(--mono);letter-spacing:.14em;text-transform:uppercase;color:var(--violet-ink)}
.sechead h2{font:700 32px/1.1 var(--display);letter-spacing:-.005em}
.lede{color:var(--ink-2);max-width:68ch}
h3{font:700 21px/1.2 var(--sans)}
.summary{background:var(--black);color:#fff;border-radius:12px;padding:22px 24px;display:flex;flex-direction:column;gap:12px}
.summary h3{color:var(--violet)}
.summary ol{margin:0;padding-left:22px;display:flex;flex-direction:column;gap:7px;color:#e6e6e6}
.summary b{color:#fff}
.stats{display:grid;grid-template-columns:repeat(auto-fit,minmax(190px,1fr));gap:12px}
.stat{background:var(--card);border:1px solid var(--line);border-radius:10px;padding:14px 16px;display:flex;flex-direction:column;gap:3px}
.stat b{font:700 28px/1.1 var(--display)}
.stat span{color:var(--ink-2);font-size:14.5px}
.stat.bad b{color:var(--crit)} .stat.good b{color:var(--green-ink)}
.finding{background:var(--card);border:1px solid var(--line);border-radius:12px;padding:20px 22px;display:flex;flex-direction:column;gap:14px;break-inside:avoid}
.fhead{display:flex;gap:10px;align-items:center;flex-wrap:wrap}
.pill{font:500 10.5px var(--mono);letter-spacing:.1em;text-transform:uppercase;padding:4px 9px;border-radius:999px;white-space:nowrap}
.crit{background:var(--crit-soft);color:var(--crit)} .high{background:var(--high-soft);color:var(--high)}
.med{background:var(--med-soft);color:var(--med)} .ok{background:var(--green-soft);color:var(--green-ink)}
.chip{font:500 10.5px var(--mono);letter-spacing:.08em;text-transform:uppercase;padding:4px 9px;border-radius:999px;border:1px solid var(--line);color:var(--ink-2);background:#fafafa;white-space:nowrap}
.chip.cat{border-color:var(--violet);color:var(--violet-ink);background:var(--violet-soft)}
.chip.origin{border-color:var(--ink-2);color:var(--ink)}
.where{font:400 12.5px var(--mono);color:var(--ink-2)}
dl.row{display:grid;grid-template-columns:86px 1fr;gap:8px 14px;margin:0}
dl.row dt{font:500 10.5px var(--mono);letter-spacing:.1em;text-transform:uppercase;color:var(--ink-2);padding-top:4px}
dl.row dd{margin:0}
dl.row dt.fix{color:var(--green-ink)}
figure{margin:0;display:flex;flex-direction:column;gap:8px;break-inside:avoid}
.shot{border:1px solid #cfcfcf;border-radius:10px;overflow:hidden;background:#fff}
.shot .bar{display:flex;align-items:center;gap:8px;background:#ececec;padding:7px 12px;border-bottom:1px solid #d6d6d6}
.shot .dots{display:flex;gap:5px}
.shot .dots i{width:9px;height:9px;border-radius:50%;background:#c9c9c9;display:block}
.shot .url{font:400 11.5px var(--mono);color:#555;background:#fff;border-radius:5px;padding:2px 10px;flex:1;min-width:0;white-space:nowrap;overflow:hidden;text-overflow:ellipsis}
.shot img{display:block;width:100%;height:auto}
.shot.blank .empty{aspect-ratio:1200/621;display:flex;align-items:center;justify-content:center;background:#fff}
.shot.blank .empty span{font:500 12px var(--mono);color:#b5b5b5;border:1px dashed #d9d9d9;padding:8px 14px;border-radius:6px}
figcaption{font-size:14.5px;color:var(--ink-2)}
figcaption b{color:var(--ink)}
.pair{display:grid;grid-template-columns:1fr 1fr;gap:14px}
.pair>*,.grid2>*,.flow>*{min-width:0}
.mobile .shot{background:#222}
.tablewrap{overflow-x:auto;border:1px solid var(--line);border-radius:12px;background:var(--card)}
table{border-collapse:collapse;width:100%;min-width:620px;font-size:15.5px}
th,td{text-align:left;padding:11px 13px;border-bottom:1px solid var(--line);vertical-align:top}
th{font:500 10.5px var(--mono);letter-spacing:.1em;text-transform:uppercase;color:var(--ink-2);background:#fafafa}
tr:last-child td{border-bottom:0}
td.num,th.num{text-align:right;font-variant-numeric:tabular-nums;white-space:nowrap}
tr.total td{font-weight:700;background:#fafafa}
td.bad{color:var(--crit);font-weight:600} td.good{color:var(--green-ink);font-weight:600}
.grid2{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:14px}
.card{background:var(--card);border:1px solid var(--line);border-radius:12px;padding:18px 20px;display:flex;flex-direction:column;gap:8px;break-inside:avoid}
.card h3{font-size:18px}
.card .price{font:700 24px var(--display);color:var(--violet-ink)}
.card p{color:var(--ink-2);font-size:15.5px}
ul.clean{margin:0;padding-left:20px;display:flex;flex-direction:column;gap:6px}
.hbars,.funnel{display:flex;flex-direction:column;gap:8px;margin-top:4px}
.hb,.fr{display:grid;grid-template-columns:128px 1fr 62px;gap:10px;align-items:center;font-size:14.5px}
.hl{color:var(--ink)}
.hv{text-align:right;font-variant-numeric:tabular-nums;font-weight:700}
.ht{height:14px;background:#efefed;border-radius:4px;overflow:hidden}
.ht i{display:block;height:100%;background:var(--violet-ink);border-radius:0 4px 4px 0}
.hb.hot .ht i{background:var(--crit)} .hb.hot .hl,.hb.hot .hv{color:var(--crit)}
.hb:hover .ht,.fr:hover .ht{background:#e4e4e1}
.flow{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:10px}
.step{background:var(--black);color:#fff;border-radius:10px;padding:14px;display:flex;flex-direction:column;gap:6px;break-inside:avoid}
.step .q{font:500 10.5px var(--mono);letter-spacing:.1em;text-transform:uppercase;color:var(--green)}
.step p{color:#d4d4d4;font-size:14.5px}
.step b{color:#fff}
.gantt{display:flex;flex-direction:column;gap:8px;break-inside:avoid}
.gantt .gr{display:grid;grid-template-columns:200px 1fr;gap:12px;align-items:center}
.gantt .lab{font-size:14.5px;font-weight:600}
.gantt .lab small{display:block;font:400 11.5px var(--mono);color:var(--ink-2)}
.track{position:relative;height:22px;background:repeating-linear-gradient(90deg,#e7e7e5 0 1px,transparent 1px calc(100%/12));border-radius:4px}
.track i{position:absolute;top:3px;bottom:3px;border-radius:3px;background:var(--violet)}
.track i.g{background:var(--green)} .track i.k{background:var(--ink)}
.weeks{display:grid;grid-template-columns:200px 1fr;gap:12px}
.weeks div:last-child{display:grid;grid-template-columns:repeat(12,1fr);font:400 10.5px var(--mono);color:var(--ink-2)}
.callout{border-left:4px solid var(--violet);background:var(--violet-soft);padding:14px 18px;border-radius:0 10px 10px 0;display:flex;flex-direction:column;gap:6px}
.callout.green{border-color:var(--green);background:var(--green-soft)}
.note{color:var(--ink-2);font-size:14.5px}
.closing{background:var(--black);color:#fff;padding-block:36px}
.closing .page{display:flex;justify-content:space-between;align-items:center;gap:20px;flex-wrap:wrap}
.closing h2{font:700 30px var(--display);color:var(--violet)}
.closing p{color:#cfcfcf}
.cta{display:inline-block;background:var(--green);color:#06140e;font-weight:700;padding:12px 20px;border-radius:6px;text-decoration:none}
.closing img{height:40px}
.printsign{display:none}
@media (max-width:640px){
  dl.row{grid-template-columns:1fr} dl.row dt{padding-top:6px}
  .pair{grid-template-columns:1fr}
  .gantt .gr,.weeks{grid-template-columns:1fr}
  .weeks div:first-child{display:none}
}
@media print{
  @page{size:A4;margin:0}
  body{font-size:13.5px}
  .page{max-width:none;padding-inline:14mm}
  .cover{padding-block:14mm 12mm}
  main.page{display:block;padding-block:28px 10px}
  main.page>section{display:block;margin-bottom:26px}
  main.page>section>*+*{margin-top:14px}
  .sechead{display:block;break-inside:avoid;break-after:avoid;page-break-after:avoid}
  .sechead .n{display:block;margin-bottom:4px}
  .sechead+*{break-before:avoid;page-break-before:avoid}
  h3{break-after:avoid}
  .flow{grid-template-columns:repeat(5,1fr)}
  .stats{grid-template-columns:repeat(auto-fit,minmax(150px,1fr))}
  .closing{display:none}
  .printsign{display:flex;justify-content:space-between;align-items:center;background:var(--black);color:#cfcfcf;border-radius:10px;padding:12px 18px;break-inside:avoid;font-size:12px}
  .printsign img{height:28px}
  html,body{height:auto}
  .pagebreak{break-before:page}
  .tablewrap{overflow:visible} table{min-width:0;font-size:12.5px}
  tr{break-inside:avoid}
}
@media (prefers-reduced-motion:reduce){*{transition:none!important;animation:none!important}}
```

### 11.2 build.py

Inlines fonts and images so the page depends on nothing external.

```python
import base64, re, os
R = os.path.dirname(os.path.abspath(__file__))
F = R + '/fonts/'

def b64(p):
    return base64.b64encode(open(p, 'rb').read()).decode()

faces = [
    ('Roboto Condensed', 400, 'fontsource-roboto-condensed-5.3.0/package/files/roboto-condensed-latin-400-normal.woff2'),
    ('Roboto Condensed', 600, 'fontsource-roboto-condensed-5.3.0/package/files/roboto-condensed-latin-600-normal.woff2'),
    ('Roboto Condensed', 700, 'fontsource-roboto-condensed-5.3.0/package/files/roboto-condensed-latin-700-normal.woff2'),
    ('JetBrains Mono', 400, 'fontsource-jetbrains-mono-5.3.0/package/files/jetbrains-mono-latin-400-normal.woff2'),
    ('JetBrains Mono', 500, 'fontsource-jetbrains-mono-5.3.0/package/files/jetbrains-mono-latin-500-normal.woff2'),
]
ff = ''.join(
    f"@font-face{{font-family:'{n}';font-weight:{w};font-style:normal;font-display:swap;"
    f"src:url(data:font/woff2;base64,{b64(F + p)}) format('woff2')}}\n"
    for n, w, p in faces)
css = ff + open(R + '/src/brand.css').read()
logo = 'data:image/png;base64,' + b64(R + '/img/ecorn-logo.png')
os.makedirs(R + '/out', exist_ok=True)

for name in ['report']:
    s = open(R + f'/src/{name}.html').read().replace('{{CSS}}', css).replace('{{logo}}', logo)
    s = re.sub(r'\{\{img:([\w-]+)\}\}',
               lambda m: 'data:image/jpeg;base64,' + b64(R + f'/img/{m.group(1)}.jpg'), s)
    assert '{{' not in s
    open(R + f'/out/{name}.html', 'w').write(s)
    head, body = s.split('</style>', 1)
    open(R + f'/out/{name}-full.html', 'w').write(
        '<!doctype html><html lang="en"><head><meta charset="utf-8">'
        '<meta name="viewport" content="width=device-width,initial-scale=1">'
        + head + '</style></head><body>' + body + '</body></html>')
    print(name, os.path.getsize(R + f'/out/{name}.html') // 1024, 'KB')
```

Screenshot prep:

```python
from PIL import Image
im = Image.open(src).convert('RGB')
im = im.resize((1200, round(im.height * 1200 / im.width)), Image.LANCZOS)
im.save('img/<name>.jpg', quality=74, optimize=True)
```

### 11.3 render.py

```python
from playwright.sync_api import sync_playwright
R = '<absolute path>/out/'
with sync_playwright() as p:
    b = p.chromium.launch()
    for n in ['report']:
        pg = b.new_page(viewport={'width': 1280, 'height': 900})
        pg.goto('file://' + R + n + '-full.html'); pg.wait_for_timeout(800)
        pg.pdf(path=R + n + '.pdf', format='A4', print_background=True, prefer_css_page_size=True)
        m = b.new_page(viewport={'width': 390, 'height': 800})
        m.goto('file://' + R + n + '-full.html')
        print(n, 'mobile overflow', m.evaluate('document.documentElement.scrollWidth'))  # must print 390
    b.close()
```

Check the PDF before delivering: `pdftoppm -r 45 -png out/report.pdf out/p`, stitch into one montage, look for orphan headings, half-empty pages and split tables. Fix with `.pagebreak` or by trimming copy.

## 12. Deliver

Publish `out/report.html` as a hosted page (or send the HTML file), send the PDF, then save a summary to your notes with the page link, headline numbers, the Stage 2 estimate total and open questions. Close every Chrome tab opened during the audit.

## 13. Honesty rules

- Only claim what was observed. If a bug did not reproduce on a second attempt, drop it.
- Orders and revenue come from Shopify; GA4 explains source and behaviour; Clarity counts and recordings are behaviour evidence, not sales.
- Never compare GA4 periods across a tracking change, and say which window was used.
- Mark origin as Likely unless confirmed in code or reproduced.
- Name the audience effect when comparing landing pages: homepage visitors are often returning or brand-aware.
- Flag seasonality when the comparison period is a different season.
- Say plainly what was not tested or not granted (checkout completion, ESP, other devices, data outside GA4 retention).
- In low-volume mode, say that small order counts make every percentage noisy.
- Flag contradictions between what the client said and what the store shows, and between this audit and any companion document.
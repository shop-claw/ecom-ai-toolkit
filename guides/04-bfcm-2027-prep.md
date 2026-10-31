# BFCM 2027 prep: a data-driven plan built on your connectors

Black Friday is November 26, 2027. Cyber Monday is November 29, 2027.

Most BFCM plans start from opinions and last year's screenshots. This one starts from your own data. You connect Shopify, Meta Ads, Google Ads and Klaviyo to Claude, pull what actually happened, reconcile it into one data pack, and only then build the strategy and the activities. Every plan, number and prompt below is meant to run on real connector data, not on benchmarks from memory.

The planning framework (offer, existing customers, site, channels, support, post-purchase, measurement) is adapted from Nik Sharma's [BFCM 2026 planning guide](https://www.nik.co/guides/bfcm-2026). The structure follows his questions; the data workflow, the 2027 timeline and all prompts are new. His caveat applies to everything here: past results are not forecasts.

The core premise: **start with an offer you can explain and fulfill profitably.**

## What you need

Connect these first. Each has its own guide.

| Source | Role | Guide |
|---|---|---|
| Shopify | Source of truth for orders, revenue, inventory, discounts | [Shopify connector](03-shopify-connector.md) |
| Meta Ads | Paid social spend, creative and audience performance | [Meta Ads connector](02b-meta-ads-connector.md) |
| Google Ads | Search and Shopping demand, impression share | [Google Ads connector](02a-google-ads-connector.md) |
| Klaviyo | Email and SMS audience, flows, campaign revenue | [Klaviyo connector](03b-klaviyo-connector.md) |
| GA4 (optional) | Sessions, landing pages, on-site funnel | your analytics setup |

Rules for the whole project:

- **Read-only until the plan is approved.** Connect Meta, Google and Klaviyo in read-only mode for the data pull.
- **Stage, never launch.** Meta campaigns are created paused, Google changes go through preview then confirm, Klaviyo work stays in draft, Shopify discounts are built and tested before the start time. A human presses every live button.
- **Keep everything in one place.** Save outputs in a `bfcm-2027/` folder in your [store brain](01-store-brain.md) so next year starts with this year's playbook.
- **Mark every number** as `measured` (from a connector), `derived` (calculated from measured numbers) or `assumed` (a guess you state out loud). Ask Claude to do the same.

## Timeline

Dates are weeks before Black Friday (Nov 26, 2027). Work backward from your slowest dependency, usually inventory or creative approval.

| When | Date | What must be done |
|---|---|---|
| **Now to Dec 2026** | Nov 13 to Dec 3, 2026 | Capture BFCM 2026 live with daily snapshots (Part 6). It becomes your freshest baseline. |
| Dec 2026 | Dec 7 onward | Post-mortem on BFCM 2026. Write the playbook into the brain. |
| T-16 | Aug 6, 2027 | Data pack refreshed. Offer candidates shortlisted against margin and stock. |
| T-12 | Sep 3 | Offer locked. Channel budgets and audience plan approved. Creative briefs out. |
| T-8 | Oct 1 | Creative approved. Site frozen except fixes. Email and SMS drafted in Klaviyo. |
| T-6 | Oct 15 | Discounts and bundles built and tested with realistic carts. Meta and Google campaigns built paused or staged. |
| T-4 | Oct 29 | Mobile checkout test end to end. Support answers written. All links verified. |
| T-2 | Nov 12 | Final QA. Owners confirmed. Change log opened. Daily snapshot job ready. |
| Week of | Nov 22 | Monitor. No new apps, no checkout changes. |
| After | Dec 1 onward | Review against plan. Follow the new-customer cohort. Save the playbook. |

## Part 1: Connect and verify

Before any analysis, prove each connector returns correct numbers.

**Prompt: connector check**

```
Run a connector check for [brand]. For each connected source (Shopify, Meta Ads, Google Ads, Klaviyo)
return: which account or store you can see, the date range of data you can reach, and one headline
number for [last 30 days]: Shopify net sales and orders, Meta spend and purchases, Google Ads cost and
conversions, Klaviyo email-attributed revenue.
Then list anything you cannot access and what permission is missing. Do not guess missing data.
```

Compare each headline number with the platform dashboard by eye. If they disagree, stop and fix it before going on.

## Part 2: Pull the data, source by source

Use two windows. Define them once and reuse them in every prompt.

- **Window A: last BFCM.** From 14 days before Black Friday to 3 days after Cyber Monday. For 2025 that is Nov 14 to Dec 4, 2025. For 2026 it is Nov 13 to Dec 3, 2026.
- **Window B: baseline.** The 8 weeks before Window A. This shows what BFCM changed.

Save each pull as a CSV in `bfcm-2027/data/`.

### 2.1 Shopify: what sold, at what margin, with how much stock

Shopify is the source of truth for money and inventory.

**Prompt: Shopify pull**

```
Using the Shopify connector, pull for Window A and Window B and save as CSV:
1. Net sales, orders, units, AOV and discounts by day.
2. Sales and units by product and variant, with cost per item if available, so we can compute gross margin.
3. Discount code and automatic discount performance: orders, revenue, discount value, and the share used by returning customers.
4. New vs returning customer orders and revenue by day.
5. Sessions, add to cart, reached checkout and completed checkout (ShopifyQL), by device.
6. Sessions and sales by referrer and by landing page.
7. Refunds and returns for the orders in Window A, by product.
8. Current inventory on hand and available per product, and the average daily sales rate in Window B.
9. Subscription orders vs one-time orders if subscriptions exist.
Flag anything you could not pull. Mark each table measured or derived.
```

What to look for: which products carried the revenue, which sold out and when, which discount mechanics leaked margin, and which products returned the most.

### 2.2 Meta Ads: what it cost to reach people and which creative held up

**Prompt: Meta pull**

```
Using the Meta Ads connector in read-only mode, for each ad account in scope pull for Window A and Window B:
1. Spend, impressions, CPM, CTR, purchases, purchase value, CPA and ROAS by day.
2. The same metrics by campaign and ad set, with objective and audience type (prospecting, retargeting).
3. Top 15 ads by spend with their creative, CPA and ROAS, and frequency at the end of each flight.
4. How CPM changed from Window B to the peak days of Window A.
5. Spend and ROAS by placement and by country.
List which campaigns hit budget limits or learning limits. Note that Meta-reported purchase value is
attributed and will not match Shopify revenue exactly.
```

What to look for: how much CPM rose at the peak, which creative fatigued, what retargeting audience size ran out, and how much spend landed in the last 3 days before the event.

### 2.3 Google Ads: how much demand you missed

Use the read-only GAQL tool on the official server. Example queries (adjust the dates and verify field names against the current API version):

```sql
-- Daily performance by campaign
SELECT segments.date, campaign.name, campaign.advertising_channel_type,
       metrics.cost_micros, metrics.clicks, metrics.conversions, metrics.conversions_value
FROM campaign
WHERE segments.date BETWEEN '2025-11-14' AND '2025-12-04'

-- Where budget or rank limited you
SELECT campaign.name, metrics.search_impression_share,
       metrics.search_budget_lost_impression_share, metrics.search_rank_lost_impression_share
FROM campaign
WHERE segments.date BETWEEN '2025-11-14' AND '2025-12-04'
  AND campaign.advertising_channel_type = 'SEARCH'

-- Search terms that sold
SELECT search_term_view.search_term, campaign.name, metrics.cost_micros,
       metrics.conversions, metrics.conversions_value
FROM search_term_view
WHERE segments.date BETWEEN '2025-11-14' AND '2025-12-04'
ORDER BY metrics.conversions_value DESC
LIMIT 100
```

**Prompt: Google pull**

```
Using the Google Ads read tools, pull for Window A and Window B: spend, clicks, conversions and
conversion value by day and campaign; brand vs non-brand split; impression share lost to budget and
to rank for each search campaign; the top 100 converting search terms; and Shopping or Performance Max
spend and value if present. Save as CSV. Then list the campaigns that were budget-limited on the
highest-revenue days, with an estimate of the demand left on the table. Mark that estimate as assumed.
```

What to look for: budget-limited brand and high-intent campaigns on the biggest days, and the search terms worth pre-building ads and landing pages for.

### 2.4 Klaviyo: who you can reach and what they did

**Prompt: Klaviyo pull**

```
Using the Klaviyo connector in read-only mode, pull:
1. Every campaign sent in Window A and Window B: send time, audience, recipients, open rate, click rate,
   revenue, revenue per recipient, unsubscribe rate and spam complaint rate.
2. Flow performance for Window A vs Window B for welcome, abandoned cart, browse abandonment,
   post-purchase and winback: recipients, conversion rate and revenue.
3. Current segment sizes: engaged in the last 30, 90 and 180 days, VIP or top customers, recent buyers
   (last 30 days), and unengaged.
4. List growth per week for the last 12 months.
5. Send frequency per profile in Window A and the unsubscribe rate by frequency band.
Note that Klaviyo-attributed revenue overlaps with Meta and Google and cannot be added to them.
```

What to look for: which segment produced the most revenue per recipient, at what frequency unsubscribes jumped, and how large the engaged audience is today, because that sets the realistic email revenue ceiling.

## Part 3: Reconcile into one data pack

Four platforms will each claim credit for the same order. Resolve that before planning.

Rules:

1. **Shopify decides revenue, orders and margin.** Everything else explains where they came from.
2. **Never add platform-attributed revenue.** Meta, Google and Klaviyo overlap.
3. Use **blended numbers** for budget decisions: total net sales divided by total ad spend (MER), and new-customer revenue divided by acquisition spend.
4. Keep the platform numbers for within-platform decisions: which ad, which search term, which email.

**Prompt: reconcile**

```
Build the BFCM data pack for [brand] from the CSVs in bfcm-2027/data/.
Produce one table by day for Window A: Shopify net sales, orders, new-customer orders, gross margin,
total ad spend (Meta + Google), blended MER, Meta-reported ROAS, Google-reported ROAS,
Klaviyo-attributed revenue, emails sent, sessions.
Then answer:
1. Total attributed revenue across platforms vs Shopify net sales. How much overlap is there?
2. On which days did blended MER fall below [target] and what changed?
3. What share of revenue came from new vs returning customers, and what was the margin of each?
4. Which products sold out, on what day, and what did that cost us (estimated, marked assumed)?
Mark every figure measured, derived or assumed. List the data gaps that limit confidence.
Save the pack as bfcm-2027/data-pack.md.
```

Read the pack yourself before moving on. If a number surprises you, ask Claude to show the query behind it.

## Part 4: Build the strategy from the data

Each prompt below needs the data pack attached or referenced. They map to the seven planning questions.

### 4.1 Choose the offer

- Start from offers your customers already responded to.
- Treat the sale as a deadline that adds pressure on inventory, margin and staff.
- Compare real options: bundle, gift with purchase, limited set, selected-product discount. A bigger discount is not automatically better.

**Prompt: offer economics**

```
Using the data pack and product-level Shopify data, compare these offer options for BFCM 2027:
[bundle A], [gift with purchase B], [limited set C], [X% off selected products D].
For each, compute from our own history: units we can sell given inventory and the daily sell rate,
gross margin per order after the offer, discount leakage to returning customers, and the effect on AOV.
Rank by expected contribution margin, not revenue. Use only our data; where you must assume, label it
assumed and show the sensitivity. Do not use industry benchmarks unless you cite a source and date.
```

### 4.2 Protect existing customers and subscribers

- Check the offer from an existing customer's side before you publish.
- Make sure a subscriber cannot cancel, rebuy at the public deal and come out ahead.
- Reward loyalty on purpose: early access, a gift, an exclusive bundle.
- Suppress purchase messages to people who just bought that product.

**Prompt: loyalty stress test**

```
Here is the BFCM 2027 offer: [terms]. Using Shopify order history, subscription rules and the Klaviyo
VIP and recent-buyer segments, find every way an existing customer could end up better or worse off
than a new one. Size each loophole using last year's returning-customer discount usage. Propose one
early-access mechanic for the top [N] customers and estimate its cost. Draft the suppression rules
for Klaviyo as a list of segment definitions I can review. Do not create anything yet.
```

### 4.3 Plan channels together

This is where the four connectors earn their keep. Plan budget, audience and creative as one system.

**Prompt: cross-channel plan**

```
Using the data pack, build a BFCM 2027 channel plan.
Inputs: total paid budget [amount], target blended MER [x], email list engaged segments today, last
year's daily revenue curve, and Meta CPM change from baseline to peak.
Return:
1. A daily budget curve for Meta and Google from T-14 to T+3, with the reasoning, and how much to hold
   back for the peak days where Google was budget-limited last year.
2. Meta: prospecting vs retargeting split, the audience sizes needed (check against the retargeting
   pool we have today), and creative volume needed to avoid the fatigue seen last year.
3. Google: campaigns to lift budget on, search terms to pre-build, brand protection plan.
4. Klaviyo: send calendar by segment and frequency cap, based on the unsubscribe-by-frequency data.
5. The handoffs: what email learns from Meta creative, what Google brand search gets from email sends.
6. Three scenarios (low, base, high) for revenue and margin, every input labeled measured or assumed.
```

### 4.4 Stage the activities, never launch them

Once the plan is approved, let Claude build the work in a paused or draft state through the connectors.

| Channel | What Claude may do | What stays human |
|---|---|---|
| Meta | Create campaigns, ad sets and ads **paused**, using dry run first | Activating, budget changes |
| Google Ads | Stage campaigns, keywords, negatives and budgets, then show the preview | `confirm_change` and enabling |
| Klaviyo | Draft campaigns, templates and flow changes | Audience check, send and schedule |
| Shopify | Prepare discounts and bundles in a test setup, list the test carts | Publishing and the schedule |

**Prompt: stage the build**

```
Build the approved BFCM 2027 plan in staging only.
Meta: paused campaigns per the plan, dry run first, naming convention BFCM27 | [objective] | [audience].
Google: staged changes with preview, no confirm.
Klaviyo: drafts only, each with its audience, exclusions, links and the discount code it uses.
Return a launch sheet: every object created, its ID, its status, who approves it and when it goes live.
Anything you could not stage, list with the reason.
```

### 4.5 Site, support and post-purchase

- Show the same offer on every page, ad and message.
- State exclusions. Test discount stacking with realistic carts. Check the schedule and timezone, and the expired-offer state.
- Walk the mobile path from ad link to order confirmation.
- Write support answers before launch, and promise only what the team can honor.
- Time post-purchase messages to delivery and use, not to the order confirmation.

**Prompt: readiness checklist from your own data**

```
Create the pre-launch QA checklist for BFCM 2027 using our data: the 5 most common cart compositions
from last year's Window A orders (from Shopify), the top payment methods and countries, the discount
combinations that caused problems, and the pages that drove the most sessions. Output a table:
check, how to test, expected result, owner, status.
Then draft the support playbook: the 20 most likely questions, approved answers, escalation owners,
and macros for stockouts, delays and discount disputes. Flag any answer that depends on a policy
I have not given you.
```

**Prompt: post-purchase sequence**

```
Design the post-purchase email and SMS sequence for first-time BFCM 2027 buyers. Use last year's data:
median delivery time, time to first reorder by product, return rate by product, and the flow
performance in the data pack. For each message: trigger relative to delivery, goal, content, and the
metric that tells us it worked. Draft in Klaviyo as drafts only.
```

## Part 5: Guardrails during build and launch

- One owner for the offer, the site, each ad platform, Klaviyo, inventory and customer communication. Write the names down.
- A change log from T-2: what changed, when, why, who approved.
- Freeze rule: from Nov 22, no new apps, no checkout changes, no new theme code.
- Keep Meta budget caps and Google budget limits set in the connector config, so a typo cannot multiply spend.
- Use the connectors' read-only switch whenever you are only analyzing.

## Part 6: Monitor live with daily snapshots

During the event, run the same snapshot every morning and mid-afternoon, and save each as a file. This is also how you capture BFCM 2026 for the baseline.

**Prompt: daily snapshot**

```
Run the BFCM snapshot for [date] using Shopify, Meta, Google Ads and Klaviyo, read-only.
Return: net sales, orders, AOV and margin vs plan and vs the same day last year; new vs returning;
blended MER and spend by channel vs the budget curve; any campaign that is budget-limited or has
frequency above [x]; Klaviyo sends today with revenue so far, unsubscribe and complaint rates;
products below [N] days of stock at today's sell rate; site conversion by device.
End with the three decisions I need to make in the next 2 hours, each with the data behind it.
Do not change anything. Save as bfcm-2027/snapshots/[date]-[am|pm].md.
```

Decisions to make from the snapshot, by a person: lift or cut a budget, swap a creative, hold or pause a send, remove a sold-out product from ads.

## Part 7: After the event: post-mortem and playbook

**Prompt: post-mortem**

```
Run the BFCM post-mortem for [year] from the daily snapshots and a fresh pull from all four connectors.
Return: results vs plan by channel and offer, margin after returns, what surprised us, what to keep,
what to drop, the change log in order, new-customer cohort size, and the plan for following that cohort
(first reorder timing, products, expected repeat rate marked assumed).
Write a one-page playbook for next year. Label every figure measured, derived or assumed.
Save to bfcm-2027/postmortem.md and update the store brain's playbooks.
```

Then, at day 30, 60 and 90, pull the cohort again and compare against the post-mortem. The discount did not create loyalty by itself; the data shows whether it did.

## One-page sale plan (fill this in from the data pack)

1. The offer, eligible products, price, inventory, start and end times, and the margin per order.
2. The effect on existing customers and subscribers.
3. Budget and audience plan by channel, with the daily curve.
4. Klaviyo calendar by segment with a frequency cap.
5. Every page, ad, message and partner link that must carry the same terms.
6. Test results for mobile checkout, owners, and the monitoring schedule.
7. Support answers, post-purchase sequence, and the review metrics.

## Questions teams ask

- **Do we need a sitewide discount?** No. Choose by margin, inventory and what your own data shows.
- **What if a connector is missing?** Run the plan on the sources you have, and list the gap in the data pack. Do not fill it with a guess.
- **Which number do we trust?** Shopify for revenue and margin. Platform numbers only inside their own platform.
- **Can Claude launch the campaigns?** It can stage them. A person launches. That rule is what makes it safe to give an AI write access in November.

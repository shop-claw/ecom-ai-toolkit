# BFCM 2027 prep guide, with prompts

Black Friday is November 26, 2027. Cyber Monday is November 29, 2027.

The planning framework here is adapted from Nik Sharma's [BFCM 2026 planning guide](https://www.nik.co/guides/bfcm-2026). The structure follows his seven questions; the wording, the 2027 timeline and all the prompts are new. Read the original too. Its own caveat applies: past results and discount ranges are not forecasts for your store.

The core premise: **start with an offer you can explain and fulfill profitably.**

## How to use this guide

Work through the seven sections with your team. Each ends with a prompt you can paste into Claude, with your data attached. Replace everything in `[brackets]`.

## Timeline (weeks before Black Friday, Nov 26, 2027)

| When | Date (approx.) | What must be done |
|---|---|---|
| T-16 | Aug 6 | Offer candidates shortlisted. Inventory and supplier commitments checked. |
| T-12 | Sep 3 | Offer locked. Creative and creator briefs started. Landing page plan approved. |
| T-8 | Oct 1 | Creative approved. Site changes frozen except fixes. Email and SMS drafted. |
| T-6 | Oct 15 | Discounts and bundles built in staging and tested with realistic carts. |
| T-4 | Oct 29 | Mobile checkout test, end to end. Support answers written. Partner links verified. |
| T-2 | Nov 12 | Final QA. Owners and escalation paths confirmed. Change log opened. |
| Week of | Nov 22 | Monitoring. No new apps, no checkout changes. |
| After | Dec 1 on | Review against plan. Start the new-customer cohort. Save the playbook. |

Work backward from your slowest dependency, usually inventory or creative approval, and move these dates.

## 1. Choose the offer

- Start from offers your customers already responded to this year.
- Treat the sale as a deadline that adds pressure on inventory, margin and staff.
- Compare real options: bundle, gift with purchase, limited set, selected-product discount. A bigger discount is not automatically better.
- For each option, write down price, conditions, inventory and fulfillment cost.

**Prompt: offer comparison**

```
You are a DTC ecommerce strategist. Compare these Black Friday 2027 offer options for [brand].
Attached: last 12 months of orders, product margins, current inventory, and past promotion results.

Options: [bundle A], [gift with purchase B], [limited set C], [X% off selected products D].

For each option return a table: price, qualifying conditions, units of inventory needed,
fulfillment cost per order, gross margin per order after the offer, and the main risk.
Then rank them by expected contribution margin, not revenue. Say what data is missing
and where you are guessing. Do not use benchmark numbers unless you cite the source.
```

## 2. Protect existing customers and subscribers

- Look at the offer from an existing customer's side before you publish.
- Make sure a subscriber cannot cancel, rebuy at the public deal and come out ahead.
- Reward loyalty on purpose: early access, an exclusive bundle, a gift, a price benefit.
- Tell support how to handle people who subscribed or bought just before the sale.
- Suppress purchase messages to people who just bought that product.

**Prompt: loyalty stress test**

```
Here is our Black Friday 2027 offer: [offer terms]. Here are our subscription rules and
price history for the last 90 days: [data].

Find every way an existing subscriber or a recent buyer could end up better or worse off
than a new customer. For each loophole return: the steps the customer takes, the cost to us,
and a fix (rule change, eligibility wording, or support script). Then propose one
early-access or gift mechanic that rewards loyalty without discounting below [floor].
```

## 3. Get the website ready

- Show the same offer on the homepage, collection pages, product pages and campaign landing pages.
- State exclusions, and whether a code is needed.
- Test automatic discounts and combinations with realistic carts.
- **Schedule check:** start and end time, timezone, before and after prices, and what the site shows after expiry. Emails and discounts must match.
- **Mobile test:** ad link to order confirmation, with every payment method, shipping charge, bundle and subscription choice.
- Check page speed. Do not launch an untested app or a major checkout change during the sale.

**Prompt: site audit checklist**

```
Create a pre-launch QA checklist for our Black Friday 2027 sale on Shopify.
Offer: [terms]. Start: [date time timezone]. End: [date time timezone].
Pages that must carry the offer: [list]. Discount stack rules: [rules].

Output a table with columns: check, how to test, expected result, owner, status.
Include: schedule and timezone, discount stacking with 5 realistic carts I will list below,
mobile path from [ad link] to confirmation, shipping rates by country, expired-offer state,
app and script changes frozen after [date], and page speed targets for [templates].
Carts: [cart 1], [cart 2], [cart 3], [cart 4], [cart 5].
```

## 4. Align ads, email and creators

- Build promotional versions of creative that already performs.
- Brief creators and partners with the exact offer, approved product facts, destination link, timing and exclusions.
- Check every link. A caption and its landing page must promise the same thing.
- Make email and SMS useful: availability, what is included, what changes during the event, when it truly ends.
- Set frequency from your own engagement data.
- Prepare alternate copy for sellouts, delivery delays and bundle confusion.

**Prompt: creator and partner brief**

```
Write a one-page brief for creators and partners for our Black Friday 2027 offer.
Facts you may use, and nothing else: [offer terms, dates, exclusions, destination URL, approved claims].
Include: what to say, what not to say, required disclosures, link and code, posting window,
and who to contact. Keep it under 300 words. Then write 3 caption variants that make exactly
the promise the landing page makes.
```

**Prompt: contingency copy**

```
Draft short email and SMS copy for these scenarios during our sale: [hero product sells out],
[carrier delays of N days], [customers confuse bundle A with bundle B], [we extend the sale].
For each: subject line, 2 sentence body, what the site banner says, and what support says.
Never promise a policy we cannot honor: [policy constraints].
```

## 5. Prepare support and operations

- Write answers to the predictable questions: stacking, purchases just before the sale, excluded products, shipping and returns.
- Promise only what the team can honor.
- Show real stock and deadlines, and tie the promotion to fulfillment capacity.
- Decide how delays are communicated and where customers check order status.
- Assign owners for the offer, site, campaigns, inventory and customer communication.
- Keep one current source of terms that everyone uses.

**Prompt: support playbook**

```
Build a support playbook for Black Friday 2027.
Terms: [offer terms, return policy, shipping times, exclusions].
Return: (1) the 20 most likely customer questions with approved answers, (2) escalation rules
and who owns each, (3) macros for delays, stockouts and discount disputes, (4) a daily
10 minute standup agenda for the sale week. Flag any answer that depends on a policy I have
not told you, instead of inventing it.
```

## 6. Plan what happens after the first purchase

- Schedule follow-up around delivery and product use, not the order confirmation.
- Send onboarding help: how to start, care for the product, fit it into a routine.
- Ask for feedback only after a reasonable time to use the product, and respond to problems.
- Make next-purchase messages relevant: replenishment timing should match the product's lifespan.
- For starter-set buyers, get them value from the set before offering a bigger bundle.
- Measure what the BFCM cohort actually does. Do not assume the discount created loyalty.

**Prompt: post-purchase sequence**

```
Design a post-purchase email and SMS sequence for first-time Black Friday 2027 buyers of [product/bundle].
Typical delivery time: [days]. Typical time to first use: [days]. Product lifespan or reorder cycle: [days].
For each message: send trigger (relative to delivery, not order), goal, subject line, key content,
and the metric that tells us it worked. Include one message that asks for feedback and one that
offers a replenishment or complementary product, each timed to real usage.
```

## 7. Measure during and after

- Define success before launch.
- Track traffic, conversion, order value, revenue, margin and the stock needed to fulfill.
- Break results down by channel and offer, so a strong total does not hide weak spots.
- Keep a change log of offer, creative and spend changes, with when and why.
- After the event, compare with plan, including returns, support volume and fulfillment problems.
- Follow the new-customer cohort for usage and repeat purchase.
- Save offers, messages, page versions and decisions as next year's playbook.

**Prompt: scorecard and post-mortem**

```
Create a BFCM 2027 scorecard for [brand]. Before the sale, define targets for: sessions, conversion,
AOV, revenue, gross margin, contribution margin, units vs stock, returns, support tickets.
Baseline data: [attached]. Show targets as a range with the reasoning.

After the sale I will paste actuals. Then produce: results vs plan by channel and offer, what
surprised us, what to keep, what to drop, the change log in order, and a one page playbook for 2028.
Mark every number as measured or estimated.
```

## One-page sale plan (fill this in first)

1. State the offer, eligible products, price, inventory, start and end times.
2. Check the economics and the experience for existing customers and subscribers.
3. List every page, ad, message and partner link that must carry the same terms.
4. Test mobile checkout and assign someone to monitor the sale.
5. Prepare support answers and messages for stock or delivery changes.
6. Write the post-purchase sequence and define the review metrics.

## Questions teams ask

- **Do we need a sitewide discount?** No. Choose by customers, inventory and economics.
- **When do we start?** Work back from inventory, creative approval, partner briefing and site testing deadlines.
- **Same offer for new and existing customers?** Maybe. Check the effect on existing buyers and make eligibility explicit.
- **Can I trust last year's numbers?** Use them as your own history, not as a forecast.

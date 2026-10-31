# How to build a Store Brain

A Store Brain is the memory of one store, kept as plain Markdown and YAML in a git repo. An AI operator reads it at the start of every run, works, and writes what it learned back. Git is the only durable state. There is no database and no daemon.

This is the pattern behind ShopClaw. You can copy it for any Shopify store in an afternoon.

## Why a brain and not a long prompt

- Chat context disappears. A repo does not.
- A repo is reviewable. Every change the agent makes to its own memory is a diff you can read and revert.
- One schema for every store means one agent can work across many stores without relearning the layout.
- The brain separates what is true (facts), what is proposed (change-sets), and what happened (decisions). Agents that mix these three make confident mistakes.

## The loop

```
observe -> analyze -> act (gated) -> measure -> learn
   ^                                              |
   +---------------- Store Brain -----------------+
```

1. **Observe:** read store state and outside signals (Search Console, GA4, ad accounts, competitors).
2. **Analyze:** compare against the audit checklist, playbooks and past decisions in the brain.
3. **Act:** always produce a reviewable change-set. Auto-apply only what the write gate allows.
4. **Measure:** read outcome data after each change.
5. **Learn:** write the outcome and an updated playbook back to the brain.

"Self-improving" here means system-level: memory-conditioned decisions, playbooks edited from measured outcomes, and autonomy that widens as the track record proves out. The model itself does not retrain.

## Step 1: create the repo

One private repo per store, named `<store>-brain`. Private matters: it will hold store facts, plans and results.

```
<store>-brain/
  config.yaml          accesses manifest and autonomy gate (no secrets)
  MEMORY.md            durable store facts the agent curates
  USER.md              owner and brand profile
  audits/<date>.md             point-in-time snapshot of SEO, catalog, conversion
  change-sets/<date>-<slug>.md   a reviewable proposal, not applied yet
  decisions/<date>-<slug>.md     applied change plus its measured outcome
  playbooks/<topic>.md           heuristics the agent wrote from outcomes
  tasks/<slug>.md                backlog: found problem -> work item -> state
  sessions/<date>.md             short per-run summary for cheap recall
  controls/control.json          operator -> agent signals (pause, approvals)
  telemetry/latest.json          headline numbers snapshotted on each measure pass
```

Keep the layout identical across stores. The moment two brains differ, you need per-store logic.

## Step 2: write MEMORY.md

L1 memory. Short and curated, not a log. Update facts in place instead of appending forever.

```markdown
# MEMORY - <store>
_Curated durable facts. Keep terse. Update when a fact changes._

## Store
- Platform, plan, currency, primary domain.
- What it really sells (the real category, not the brand name).

## Standing constraints
- Autonomy policy in force: what auto-applies, what is gated.
- Known landmines: supplier-dump descriptions, leaked discount codes, mixed languages.

## What we have learned
- One line each, linking the decision or playbook that proved it.
```

## Step 3: write USER.md

L2 memory about the human. It stops the agent from re-asking and re-litigating.

```markdown
# USER / BRAND - <store>

## Owner
- Role, goals, risk tolerance, how much autonomy they have granted.

## Brand voice and positioning
- Tone, do and do not, target customer, positioning against competitors.

## Preferences observed
- Decisions they made and why, so we do not argue them again.
```

## Step 4: make tasks the unit of work

Every problem the agent finds becomes a file with a state machine.

```markdown
---
type: task
slug: fix-duplicate-meta-descriptions
created: 2027-01-12
state: open        # open | in-progress | applied | queued | done | wontfix
priority: 4        # impact x effort, 1-5
source: audits/2027-01-12.md#finding-7
---
# Fix duplicate meta descriptions
- **Problem:** hundreds of products share one meta description (evidence: audit finding 7).
- **Proposed fix:** change-set 2027-01-12-meta-descriptions.
- **Reversible:** yes
- **Outcome:** decision ref once measured
```

## Step 5: set the write gate

This is the part that lets you sleep. Put it in `config.yaml`.

- **Reversible and measurable** changes may auto-apply, under a per-run cap, and only once telemetry is connected so the result can be measured.
- **Irreversible** actions (delete, archive, price changes) always wait for a human yes.
- **Publishing** is allowed only for the classes the owner listed explicitly.
- Before reverting anything, re-fetch the live value. A store has no real undo.

A minimal gate:

```yaml
autonomy:
  require_telemetry_for_auto_apply: true
  max_changes_per_run: 25
  auto_apply: [seo_title, meta_description, image_alt, product_tags]
  needs_approval: [price, delete, archive, theme_publish]
```

## Step 6: close the loop with measurement

A change that is not measured is a guess. Wire at least:

- Search Console for organic impressions and clicks by page.
- GA4 for sessions and conversion.
- Shopify itself for orders and revenue (the source of truth for money).

After each applied change, the agent writes a `decisions/` file: what changed, the metric, the window, the result, and whether the playbook should change. Persistence is proven only by reading the value back from the store.

## Step 7: run it unattended

Because the brain is just git, the runtime can be anything that can check out a repo and run an agent on a schedule: GitHub Actions on a cron, or a hosted routine. The agent commits its work back to the brain. Keep compute ephemeral and the repo as the only memory.

Practical rules that save pain:

- Keep batch sizes small enough that a run can finish and close what it opened. Tasks left "in progress" by a killed run are the most common leak.
- Only the orchestrator touches git and the queue file. Task workers never do.
- Never print environment variables through filters. Check one named variable at a time, for example `[ -n "$VAR" ] && echo "VAR present"`. A multi-line JSON key leaks through `grep`, `cut` and `sed`.
- Secrets live in the runner's secret store. Never in the brain, never in chat.

## Step 8: let the brain grow, but curate it

- `MEMORY.md` stays short. Move detail to `playbooks/` and `decisions/`.
- Recall is `grep` over structured files. Do not add a vector database until grep stops working.
- Add a `knowledge/` folder of sourced, dated benchmarks and make the rule that industry numbers come only from there, with the source and date cited. No numbers from memory.

## Checklist

- [ ] Private `<store>-brain` repo with the canonical layout
- [ ] `MEMORY.md` and `USER.md` filled in
- [ ] `config.yaml` with the autonomy gate and the list of accesses
- [ ] Secrets in the runner, none in git
- [ ] Search Console, GA4 and Shopify read access connected
- [ ] First `audits/<date>.md` produced and reviewed by a human
- [ ] First change-set applied, read back, and written to `decisions/`

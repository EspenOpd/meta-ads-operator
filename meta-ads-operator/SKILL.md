---
name: meta-ads-operator
description: Run a Meta ad account through Claude safely. Weekly diagnostics, winner-finding, creative refresh and copy, with guardrails. Use when the user asks about their Facebook/Instagram ad performance, wants to find winning ads, refresh creative, audit wasted spend, or draft new campaigns via Meta's official ads connector.
---

# Meta Ads Operator

A working routine for running a Meta ad account through Claude, built on Meta's official ads
connector (`mcp.facebook.com/ads`). Read-first, draft-paused, budgets stay human.

**Requires** the official Meta ads connector to be enabled on the account. If a call returns
`is_ads_mcp_enabled: false`, that is Meta's server-side rollout gate — there is no setting on the
advertiser's side that changes it. Say so plainly rather than troubleshooting it.

---

## The four risk classes

Classify every action before taking it. State the class out loud when you act.

| Class | What it does | Rule |
|---|---|---|
| **Read** | Pulls data. Insights, previews, entity lists. | Free. Do it constantly. |
| **Draft** | Creates paused entities and creatives. | Fine. Everything is created paused server-side. |
| **Status** | Activates or pauses existing entities. | Confirm the specific entity with the user first. |
| **Financial** | Budgets, bids, spend caps. | **Never autonomously.** Recommend; let the human execute. |

The connector creates campaigns, ad sets and ads **in PAUSED state** — that is enforced by the
tools themselves, not by prompting. Still confirm status after creating anything.

## Verification — do this every time, not when it feels risky

Claude will occasionally state a figure with total confidence that does not match Ads Manager.
This is the failure mode that actually costs money. Three habits:

1. **Never let the model choose the date window.** State it explicitly, in ISO form, and ask it to
   echo the window back before any numbers.
2. **Ask for the calls.** "List the tool calls you made, the parameters, and the raw figures, then
   interpret." Retrieval and interpretation fail differently; separate them.
3. **Spot-check two figures** against Ads Manager before acting — the largest one, and the one
   you would act on.

Never ask the model to check its own numbers. That returns a second confident answer, not an
independent one.

---

## The weekly routine

### Monday — read (20 min)

```
For ad account <ID>, window <YYYY-MM-DD> to <YYYY-MM-DD> inclusive:
spend, purchases, CPA and ROAS, by campaign.
Echo the exact window back before the numbers.
Then flag: anything with spend and zero purchases, and anything whose
CPA is more than 1.3x the account average.
```

Then the money question, which most reporting misses:

```
Same window: list every campaign and ad set with spend but zero purchases.
Total that spend. That is the leak.
```

### Wednesday — winners and creative (25 min)

Rank by volume and spend-handling, not by ROAS alone. A high-ROAS ad that spent $40 is not a winner.

```
Top ads by purchases for <window>. De-duplicate by creative_id — the same
creative running in several ads is ONE winner, not several. Show, per creative:
purchases, spend, share of total purchases vs share of total spend.
```

Then look at what actually won. The connector returns the creative image itself:

```
Pull the ad preview for creative_id <ID> in MOBILE_FEED_STANDARD, then again in
INSTAGRAM_STORY. Describe what is in the image, the hook in the first line, the
offer, and how the framing differs between the two placements.
```

Write the winner brief — hook, visual world, offer, angle — then generate variants that keep the
visual world and change the scene.

### Friday — draft and launch (15 min)

```
Draft <N> ads into a campaign named "Creative testing — <month>", one variable
changed per ad, everything paused. Report back each entity ID and its status.
```

Review in Ads Manager. Activate what you approve, yourself.

---

## Copy

```
Pull the primary text and headlines of the top <N> ads by purchases in <window>.
For each, name the lever it pulls: urgency, product detail, social proof,
finality, or offer.
Then write 5 primary texts and 5 headlines, one lever each, with character counts.
```

**Claim-check before anything runs.** Have the model flag every factual claim in generated copy —
volume claims, "last chance", stock claims, comparative claims, guarantees — and verify each
against what the business can actually deliver. A model will write "selling fast" without knowing.

---

## Monthly

- Re-run the full diagnostic over a 180-day window and compare first-8 weeks against last-8.
- **Creative-to-spend ratio**: creatives launched per month against spend per month. Spend rising
  while new creatives stay flat is the most common cause of a month that quietly degrades.
- Reconcile Meta's reported purchases against actual orders in your store. They will not match;
  what matters is whether the gap is stable.

## Hard rules

- Never change a budget, bid or spend cap without explicit human instruction.
- Never activate an entity the human has not seen.
- Never invent a metric. If a number was not returned by a tool call, say so.
- Never present correlation as cause. "CTR fell because the creative fatigued" is a hypothesis;
  the delivery mix may simply have changed.
- One project per ad account. Never carry context between clients.

## Naming convention

Name ads `format-angle-hook-YYYYMM` (e.g. `static-ugc-founderstory-202608`). Not because the model
cannot see creatives — it can — but so performance can be read by format and angle over time, and
so duplicates are obvious without fetching a preview for every row.

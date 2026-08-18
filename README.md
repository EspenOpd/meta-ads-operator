# Meta Ads Operator — a Claude Skill

A working routine for running a Meta ad account through Claude, using Meta's official ads
connector. Read-first, draft-paused, budgets stay human.

## What's in it

- The **four risk classes** (Read / Draft / Status / Financial) and what each one is allowed to do
- A **weekly routine** — Monday read, Wednesday winners and creative, Friday draft and launch
- **Verification habits** that catch the confidently-wrong numbers, which is the failure mode that
  actually costs money here
- **Copy generation** with a claim-check step
- Monthly diagnostics: creative-to-spend ratio, 180-day comparison, store reconciliation

## Install

```bash
git clone https://github.com/EspenOpd/meta-ads-operator.git
mkdir -p ~/.claude/skills
cp -r meta-ads-operator/meta-ads-operator ~/.claude/skills/
```

Or drop the `meta-ads-operator/` folder into any project's `.claude/skills/`.

Then confirm Claude can see it:

```
What skills do you have available?
```

**Requires** Meta's official ads connector (`mcp.facebook.com/ads`) enabled on your ad account.
If a call returns `is_ads_mcp_enabled: false`, that is Meta's server-side rollout gate — there is
no setting on your side that changes it.

## Verify the two corrections yourself

Do not take this README's word for either claim. Ask your own connector:

```
List every tool you have from the Meta ads connector whose name contains
image, video, preview or media. For each one, tell me what it returns.
```

```
What status does ads_create_campaign say it creates entities in?
```

## Two things most guides get wrong

**Claude can see your ad creatives.** The official connector exposes `ads_get_ad_preview`,
`ads_get_ad_images`, `ads_get_ad_videos` and `ads_get_ig_media`. The preview tool's own description
says its response includes "the ad creative image as a separate image content item". Most published
guides still say the opposite — [the full correction, with how to verify it
yourself](https://claudeadsoperator.com/blog/can-claude-see-your-facebook-ad-images).

**Paused-on-create is enforced server-side.** `ads_create_campaign` and `ads_create_ad` describe
themselves as creating entities "in PAUSED state". It is a property of the tooling, not of how you
prompt. Confirm status after creating anything anyway.

## Honest limits

- Nothing here promises a return. It saves time, produces more creative, and removes blind spots.
- The model will sometimes state a figure that does not match Ads Manager. Verify before acting.
- Rollout is account-by-account. `is_ads_mcp_enabled: false` is a server-side gate you cannot change.
- Enabled is not the same as working: an account can be MCP-enabled and still return
  `is_queryable: false`, sometimes with no usable reason attached —
  [what that flag means](https://claudeadsoperator.com/blog/ads-mcp-account-not-queryable).

## More

Written by [Espen Opdahl](https://claudeadsoperator.com/about). The blog covers what the connector
actually does, and what breaks: [claudeadsoperator.com/blog](https://claudeadsoperator.com/blog).

The full course — the report formats, the complete prompt library, and the guardrails —
is [The Claude Ads Operator](https://claudeadsoperator.com/?utm_source=github&utm_medium=skill&utm_campaign=organic).

Not affiliated with Meta or Anthropic. MIT licensed.

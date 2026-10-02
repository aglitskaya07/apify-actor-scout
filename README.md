# Apify Actor Scout

A skill for Claude that helps you pick the right Apify Actor out of thousands and shows what a
data collection job will cost before you spend real money on it.

With the Apify MCP server connected, you can ask Claude for "the last 30 posts from these 40
profiles and the top comments under each". Claude will pick an Actor and run it. What you don't
see is why it picked that Actor, whether a similar one would have done the job for a fraction of
the price, or what the run will cost until it's done. This skill adds that step.

## What it does

1. **Counts units.** Turns your request into a number of items:
   `40 profiles × 30 posts = 1,200 posts`, then `× up to 100 comments`.
2. **Shortlists Actors.** Searches the Store and keeps 2-3 candidates per platform. It checks
   usage, rating, freshness, input shape and output fields.
3. **Makes a paper estimate.** Prices each candidate with its own event list at your plan's tier.
   The price structure matters: per comment vs per post queried can flip which Actor is cheaper.
4. **Runs a pilot for cents.** Same 1-3 inputs on every candidate. Checks for empty rows, missing
   fields and data outside your date window, and works out the cost per useful item.
5. **Quotes and stops.** "~1,200 posts + ~38,000 comments, ~$21, worst case $27. Start?"
   Nothing big runs until you say yes.
6. **Reconciles.** Compares the quote with what the run actually cost.

When you collect from several platforms, it also maps every Actor's fields to one common table.
That way Instagram, TikTok, Threads and YouTube data can be analysed together.

## Install

First connect the [Apify MCP server](https://mcp.apify.com) to Claude. The skill drives Apify
through it.

**Claude.ai / Claude Desktop**

1. Download `apify-actor-scout.zip` from the
   [latest release](https://github.com/aglitskaya07/apify-actor-scout/releases/latest).
2. In Claude, open Settings and find **Skills** (under *Customize* or *Capabilities*,
   depending on your version). Skills need code execution to be turned on.
3. Upload the zip and make sure the skill is switched on.

**Claude Code**

```bash
git clone https://github.com/aglitskaya07/apify-actor-scout.git
cp -r apify-actor-scout/apify-actor-scout ~/.claude/skills/
```

Restart Claude Code. The skill applies to every project.

**Using it.** Just describe the data you need. The skill kicks in when you ask to scrape or
collect data through Apify, or ask what a collection will cost. You can also call it by name:
"use apify-actor-scout".

## Example

> I need comments from the last 3 months under posts of these 25 Instagram and TikTok accounts.
> How much will it cost?

Claude counts the posts first. Each post already shows its comment count, so the comment volume
is known before any comments are ordered. Then Claude prices two Actors per platform, pilots
them on 2 accounts each for a few cents, and comes back with one quote and one question.

## Files

```
apify-actor-scout/
  SKILL.md                      the six-step method
  references/pricing-models.md  how to estimate pay-per-event and pay-per-usage Actors
  references/field-map.md       one common table across platforms
```

## Author

[Nailia Aglitskaya](https://www.linkedin.com/in/nailia-aglitskaya-397b18107): I teach marketing
and PR teams to work with AI agents.

## License

MIT

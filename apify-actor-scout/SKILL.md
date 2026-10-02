---
name: apify-actor-scout
description: >
  Picks the right Apify Actor out of thousands and tells you what a data collection job will cost
  before any real money is spent. Turns the task into a unit count, shortlists 2-3 Actors per
  platform, builds a paper estimate from each Actor's real price list, runs a pilot for cents,
  re-quotes from what the pilot actually returned, and starts the full run only after the user
  says yes. Afterwards it compares the quote with the bill. Use whenever the user wants to
  collect social media or web data through Apify: posts, comments, profiles, search results,
  reviews, ads, across one or several platforms. Triggers: "scrape", "collect posts/comments",
  "which Apify actor", "how much will this cost", "pull data from Instagram/TikTok/YouTube/
  Threads/LinkedIn", "run this on Apify", "estimate the cost", "dry run", "pilot run".
metadata:
  version: "1.1.0"
  requires: "Apify MCP server (search-actors, fetch-actor-details, call-actor, get-actor-run, get-dataset-items)"
---

# Apify Actor Scout

The Apify Store has thousands of Actors. For the same job, two Actors can differ in price by
an order of magnitude and in quality from "works" to "returns empty rows". Your job is to find
the right one, say what the job will cost, and spend real money only after the user agrees.

Six steps, in order. Do not skip ahead to a full run.

1. **Units:** turn the task into a number of items.
2. **Candidates:** find 2-3 Actors per platform and data type.
3. **Paper estimate:** price each candidate using its own price list.
4. **Pilot:** run each candidate on 1-3 inputs for cents.
5. **Quote and stop:** re-quote from pilot facts and wait for a yes.
6. **Reconcile:** after the run, compare the quote with the bill.

Speak to the user in plain language. They are usually marketers, not engineers. Show numbers
in tables, explain any assumption in one line, and never hide a cost inside "approximately".

## 1. Units: what exactly are we buying

Apify charges per thing returned: a post, a comment, a profile, a search result, a second of
video analysed. So before looking at any Actor, write down the shape of the job:

| Question | Example |
|---|---|
| Platforms | Instagram, TikTok |
| Whose content | the brand's own account, or everyone who mentions it (search, hashtag)? |
| Starting point | 40 profile URLs / 10 keywords / 200 post URLs |
| Depth per input | last 30 posts per profile; 50 videos per keyword |
| Second level | top 100 comments per post, replies included? |
| Time window | posts from the last 90 days, or comments from the last 90 days? |
| Extras | transcripts, AI video summary, follower lists |
| Fields actually needed | text, date, likes, comment count, author |

Multiply it out: `40 profiles × 30 posts = 1,200 posts`, and `1,200 posts × up to 100 comments
= up to 120,000 comments`. Show the multiplication to the user. It is where most surprises come
from: second-level data like comments and replies grows much faster than people expect.

"Whose content" changes everything: a brand's own posts and posts that mention the brand need
different Actors and differ in volume by orders of magnitude. Settle it first.

Ask only about what you cannot infer. If the user gave links, count them. If depth is unknown,
propose a default and say it. Where you have to guess a volume (how often an account posts, how
many comments a post gets), mark the number as a guess. Step 4 replaces guesses with counts.

**Comments are cheaper to estimate after posts.** If the job is "posts, then comments under
them", the posts usually carry their comment count. A posts-only pilot (cents) gives you the real
comment volume before you order a single comment. Check in the pilot that the count is actually
there: some Actors return it only in a more detailed, pricier mode.

## 2. Candidates: finding the right Actor among thousands

Search with `search-actors` (`limit: 10`), always twice:

- **Specific:** platform + data type, e.g. `Instagram comments`, `TikTok search`, `YouTube channel`.
- **Broad:** platform only, e.g. `Instagram`. Big all-in-one Actors often beat narrow ones,
  also on the second level: price comments through the all-in-one too, not only through a
  comments-only Actor.

Look for Actors that skip a step. Some comment scrapers take an account name and a date window
directly, so you don't pay for a separate posts run or a date-filter add-on.

Never use ranking words like "best" or "top" as keywords. One search per platform and data type.
Several platforms means several searches.

Each search result already carries price, user counts, rating and input fields. Shortlist with:

| Filter | Keep | Drop |
|---|---|---|
| Status | not deprecated | deprecated |
| Monthly users | hundreds or more | a handful (nobody keeps it alive) |
| Success rate | high (`fetch-actor-details` → `stats`) | many failed runs |
| Rating | 4+ | below 4 with many reviews. Few reviews is not a verdict: keep, let the pilot decide |
| Price model | pay per event, listed openly | "contact us"; pay per usage is fine but must be piloted, its price is unknown upfront |
| Input shape | accepts what you have (URLs vs usernames vs keywords) | needs something you don't have |
| Output | returns the fields you listed in step 1 | missing a field you need |

An Actor from `apify/` (official) is a tie-breaker, not a must. Third-party Actors are often
cheaper and just as good, which is exactly why the pilot exists.

Don't judge freshness by `modifiedAt` in metadata: it changes on routine platform updates and
says nothing about maintenance. Usage and success rate say more.

Then run `fetch-actor-details` on the 2-3 finalists with `pricing`, `inputSchema`, `readme`,
`stats` and `outputSchema`. Read the input schema and README properly. They tell you:

- **What a limit limits.** Per input or per run. If `maxItems` is a total for the whole run,
  capping comments per post means one run per post, so start fees multiply. Price it that way.
- **Whether a date filter exists**, and on what: posts or comments.
- **Whether replies are included**, and whether they count as items.
- **Plan restrictions.** Some Actors return less on the Free plan (e.g. "only the top 15
  comments"). On Free, such a pilot looks like a broken Actor. Check before blaming it.
- **Alternative charge events.** If the list has two prices for the same thing (e.g. with and
  without login cookies), price the one that applies to the inputs you will actually send.

**Price shape matters more than the headline price.** Two real examples from the Store for
Instagram comments:

- Actor A: $0.0019 per comment.
- Actor B: $0.0075 per post queried (first 15 comments free) + $0.0005 per extra comment.

For 1,000 posts with ~10 comments each, A costs ~$19 and B ~$7.50. For 20 posts with 2,000
comments each, A costs ~$76 and B ~$20. Neither is "cheaper". It depends on the shape from
step 1. Always price candidates against the user's actual shape.

## 3. Paper estimate

Price every candidate with its own event list from `fetch-actor-details`. The price list is
tiered by plan. The tool tells you the user's tier (`userTier`), so use that row. On the Free tier,
prices are often about twice the Gold ones. Do not quote a price from someone else's plan.

```
cost = actor starts × start fee
     + main items × price per item
     + second-level items × price per item
     + add-ons (date filter, transcript minutes, AI summary seconds, ...) × their price
```

Add-ons are easy to miss: on some Actors a date filter or a "scrape as country" option is a
separate charge per item, and AI summaries are charged per second of video. If the user wants
an add-on, multiply it by the expected volume (e.g. 40 videos × ~45 s average × price per second).

Show a table, one row per candidate:

| Actor | Price shape | Units | Expected cost | Notes |
|---|---|---|---|---|

Give expected and worst case (all limits hit). When the input is "an account plus a date window",
no limit gets hit on its own, so set an explicit cap (e.g. max posts = expected × 1.5) and treat
it as the worst case. Then end with one question: the pilot budget.
If a scope assumption needs confirming, state it in one line right above, so a single "yes"
covers both. "Pilots on these 3 Actors will cost under $0.10 in total. OK to run them?"

Some pay-per-event Actors also charge platform usage on top of the event price, and the Actor's
pricing section says so. Pay-per-usage Actors have no price list at all. See
`references/pricing-models.md` for how to estimate both.

## 4. Pilot: spend cents, not dollars

Run each finalist with `call-actor` on the same 1-3 inputs:

- Pick representative inputs, including one awkward one (a huge account, a non-English one,
  a post with many replies).
- Keep most pilot inputs small, but run **one input at the full planned depth** (e.g. all 30
  comments on one post). Small pilots hide what happens past the first page or past the free
  allowance: an Actor that gives 15 comments free may stop at exactly 15. The limit has to go
  past both, or the pilot proves nothing about the full run.
- Input fields like max posts or max comments are your real spending control.
- Cheap volume probe: to learn how many posts an account published in the window, ask a
  step-skipping Actor for 1 comment per post. One cheap item per post gives you the post count.
- `callOptions.maxTotalChargeUsd` caps what a pay-per-event run can charge (you are never billed
  past it), but it cannot be set below $0.50. That makes it a safety net, not a pilot limit.

Then read the result with `get-dataset-items` and check:

| Check | Fail means |
|---|---|
| Items returned vs expected | 0 items on an input that is certainly not empty → reject the Actor |
| Needed fields are filled | text, date, counts are null or empty → reject or try another mode |
| Dates fall inside the window | the Actor ignores the date filter → you would pay for old data |
| No duplicates | the same item counted twice → you pay twice |
| What a limit means | 5 "per input" became 5 total, or the reverse → fix the estimate |
| Sort order | asked for top comments, got newest (0 likes) → wrong Actor or wrong mode |
| Full depth reached | the full-depth input returned fewer items than asked while the source has more → look for a stop flag (e.g. "continue on duplicates") |
| Cost per item | compute it, see below |

A call rejected by input validation (wrong date format, missing field) starts no run and costs
nothing. Fix the input and call again.

**Actual pilot cost.** The MCP run result shows items and compute units, not dollars. Compute
the cost from the price list: items returned × price + start fee. A pilot of 2 TikTok posts on
Gold tier: 2 × $0.0017 + $0.001 start = $0.0044, and the Apify bill matched to the cent.
If the user wants to check, the exact charge is in Apify Console → Runs, about a minute after
the run ends.

The number that matters is **cost per useful item**, not cost per item. If an Actor returns
100 rows and 30 are empty or out of the date window, its real price is a third higher.

## 5. Quote and stop

Re-quote from pilot facts, not from the price list alone:

> **Quote.** Actor: `x/y`. Plan: 40 profiles × 30 posts, then top 100 comments per post.
> Expected: ~1,200 posts + ~38,000 comments (from real comment counts). Cost: ~$21, worst case
> $27. Time: ~15 min. Not included: replies, transcripts. Start the full run?

Do not start until the user says yes. If they push back on cost, offer concrete trade-offs:
fewer comments per post, a shorter window, top posts only, a cheaper Actor with known gaps.

When running:

- Set the Actor's input limits to the plan. Set `maxTotalChargeUsd` to the worst-case quote
  plus ~25 % (or the $0.50 minimum on small jobs), so a runaway run stops itself.
- If the job costs more than a few dollars, run a first batch of ~10 % and check that cost
  per useful item holds before running the rest.
- Never re-run the same input "to be sure". Re-reading collected data with `get-dataset-items`
  costs next to nothing, collecting it again costs the full price. Failed runs still cost the start fee.
- If some inputs came back incomplete, rerun only those, with the setting that caused it fixed.
  The rerun pays again for the items you already have. Count it as waste in step 6.
- Long runs: start with `waitSecs: 0` and poll with `get-actor-run`.

## 6. Reconcile

After the run, report in one short table: planned vs got (items), quoted vs computed cost,
and why they differ if they do. Add one more line, the **waste share**: money spent on rows you
did not use (empty, out of window, duplicates, incomplete runs that had to be redone) divided
by the total. Aim for 5 % or less. Above that, the Actor or its settings need another look
before the next job. If the user keeps a log, add one line per job: date, Actor,
units, cost, cost per useful item. Next estimates on the same Actor should use that observed
number, not the price list.

## Several platforms in one job

Each platform needs its own Actor, and each Actor names its fields differently
(`diggCount` vs `likesCount`, `createTimeISO` vs `timestamp`). Agree on one common table
during the pilot, while the outputs are small and easy to compare, and map each Actor to it.
See `references/field-map.md`. Keep the raw datasets too: the analysis may later need a field
nobody thought of. Re-reading a dataset is nearly free, re-scraping is not. Unnamed datasets
are deleted after your plan's retention period, so export what you will need later.

## Rules that hold everywhere

- No full run without a quote and a yes.
- Price against the user's tier and the user's shape, never against a headline price.
- A pilot that fails the checks kills the candidate, however cheap it looked on paper.
- Say plainly when something cannot be estimated (e.g. a keyword search with unknown volume).
  Then propose a capped first batch.

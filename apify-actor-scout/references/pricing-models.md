# Estimating cost by pricing model

Apify Store Actors use one of two pricing models today. The third, monthly rental, was retired
on 1 October 2026. Former rental Actors now run on pay per usage.

`fetch-actor-details` with `output: { pricing: true }` returns the model and, for pay per event,
the full event list with a price for every plan tier. The user's tier comes back as `userTier`.

## Pay per event

The Actor author defines events and their prices. Typical events:

| Event | Charged per | Watch for |
|---|---|---|
| Actor start | run | a small flat fee, but it adds up on many tiny runs |
| Result / item | row written to the dataset | the main cost |
| Comment, reply, follower | each second-level item | grows fastest of all |
| Query / post queried | each input processed | can dominate when inputs are many and items per input are few |
| Add-on: date filter, sorting, country | each item, when the option is on | easy to switch on without noticing |
| Transcript | per started minute of video | round up per video, not in total |
| AI summary / description | per second of video | multiply by the average video length |

Formula:

```
cost = Σ (event count × event price at the user's tier)
```

Things to check in the pricing section and README:

- **Free items.** Some Actors include the first N items per query for free.
- **Platform usage on top.** Most pay-per-event Actors include platform usage in the event price,
  but some charge it separately. If they do, add a pilot-measured compute cost (see below).
- **Tier gap.** Free-tier prices are often about twice the Gold-tier prices or more. A quote made on
  one plan does not hold for a user on another.

Worked example, TikTok Scraper on Gold tier: $0.0017 per result + $0.001 per start.
A pilot of 2 posts: 2 × $0.0017 + $0.001 = $0.0044. The actual charge was $0.0044.

Safety net: `maxTotalChargeUsd` in `call-actor` stops billing at that amount. Minimum $0.50.

## Pay per usage

No price list. You pay for the platform resources the run consumes: compute units (CU), storage
operations, proxies. It cannot be estimated from documentation, only from a pilot:

1. Run the pilot on 1-3 inputs with small limits.
2. `get-actor-run` → `stats.computeUnits`. Divide by the number of useful items to get CU per item.
3. Multiply by the full volume and by the CU price on the user's plan (on the Apify pricing page;
   ask the user for their plan if unknown).
4. Residential proxies are billed separately per GB. If the Actor uses them, the README usually
   says so and gives its own cost estimate. Use that as a cross-check.

Pilot cost per item on pay per usage is noisier than on pay per event. Quote a wider range.

## When the volume itself is unknown

Keyword and hashtag searches return "as many as exist", up to your limit. A specific keyword over
a short window can return far fewer than the limit. Example: 10 keywords × 50 videos asked,
483 returned. The limit is then the worst case, not the expectation. Say so, and quote the worst
case as the cap.

# One table across platforms

Each Actor returns its own field names. To analyse Instagram, TikTok, Threads and YouTube data
together, map every Actor to one common table during the pilot. At that point the outputs are
a few rows each and differences are easy to see.

## Common table

| Field | Meaning |
|---|---|
| `platform` | instagram / tiktok / threads / youtube / ... |
| `item_type` | post / video / comment / reply / profile |
| `id` | the platform's own id |
| `url` | link to the item |
| `parent_id` | for comments and replies: the post or comment they belong to |
| `author_handle` | username |
| `author_followers` | if the Actor returns it |
| `published_at` | ISO date-time, UTC |
| `text` | caption, comment or title, as is |
| `likes` | likes / hearts / diggs |
| `comments` | comment count |
| `shares` | shares / reposts |
| `views` | plays / views, where the platform has them |
| `source_actor` | which Actor produced the row |
| `source_input` | which input (URL, keyword) produced it |

Add only what the analysis needs. Don't add fields "just in case": the raw dataset already
keeps everything.

## How to build the mapping

1. After each pilot, list the fields the Actor actually filled (the `get-actor-run` result lists
   available fields; `get-dataset-items` with `fields=` shows values).
2. For each common field, name the source field per Actor, e.g. TikTok `diggCount` → `likes`,
   `createTimeISO` → `published_at`, `authorMeta.name` → `author_handle`.
3. Note gaps explicitly: if one platform has no view counts, the field stays empty for that
   platform. Do not fill it with zeros, or averages across platforms will lie.
4. Normalise dates to UTC and numbers to integers ("1.2K" → 1200) at mapping time.
5. Show the user the mapping as a small table before the full run. Gaps found now are cheap,
   gaps found after the full run mean paying to collect again.

## Keep the link back

`source_actor` and `source_input` make every row traceable. When a number looks wrong in the
analysis, you can go back to the exact run and input that produced it.

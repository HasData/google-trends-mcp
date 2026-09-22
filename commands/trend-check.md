---
description: Whether interest in a term is rising or falling, with the regions and the related queries
---

Check the trend behind a term.

Ask me for the term and the market if I have not given them, and for the window if it matters. Windows are Google's own strings such as `today 12-m` or `now 7-d`.

Then:

1. Call `hasdata_google_trends_search_getTrendsData` with `q`, `geo` and `date`, leaving `dataType` at the series. Set `tz` when the day boundary matters, because the default is UTC-7.
2. If the series comes back empty or flat at zero, say the term has no measurable interest in that market and stop. That is a real answer, and the remaining calls will not improve it.
3. Describe the shape, naming where it starts, where it ends and when the peaks happened. Say the values are relative to the window's own maximum and are not search counts. Drop or mark the last point when it carries `isPartial`, because a partial bucket looks like a decline.
4. Call again with `dataType: geoMap`, passing `region` explicitly, and report the strongest regions along with the granularity you asked for.
5. Call again with `dataType: relatedQueries` and list the rising queries separately from the top ones, because rising is about change and top is about volume. Sort the rising list on the `value` label rather than on `extractedValue`, and group the `Breakout` entries on their own instead of ranking them, since their numbers are sentinels.

Never compare a number from one call to a number from another. Each call rescales to its own maximum, so two 100s mean different things. To compare terms, pass up to five of them comma-separated in a single `q` and read the `values` array on each point. That trick does not work on the related blocks, which take one term only.

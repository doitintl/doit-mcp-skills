# Metrics and Query Options

Options for the `config` object of `run_query`.

## Metrics

`metrics` is an array of up to four objects. `metric` (a single object) is deprecated.

| Type | Value | Use for |
|------|-------|---------|
| `basic` | `cost` | Cloud cost; the default |
| `basic` | `usage` | Usage quantities |
| `basic` | `savings` | Commitment and reservation savings |
| `extended` | for example `amortized_cost` | Extended metrics |
| `custom` | the custom metric's ID | Customer-defined metrics |

## Aggregation

`aggregation`: `total` (default), `percent_total`, `percent_col`, `percent_row`.

## Time Range

`timeRange.mode`:

| Mode | Fields | Meaning |
|------|--------|---------|
| `last` | `amount`, `unit`, `includeCurrent` | The last N calendar periods. `includeCurrent: true` counts the current partial period within N |
| `current` | `unit` | The current calendar period |
| `custom` | none | Explicit dates, given in a sibling `customTimeRange` with RFC 3339 `from` and `to` |

`unit`: `day`, `week`, `month`, `quarter`, `year`. The default is the last 7 days including today.

## Time Interval

`timeInterval` sets the granularity of the time columns: `hour`, `day` (default), `week`, `isoweek`, `month`, `quarter`, `year`, plus `dayCumSum` and `week_day`. Match it to the range: daily rows over a year produce a very large result.

## Data Sources

`dataSource`: `billing` (default), `bqlens`, `billing-datahub`, `kubernetes-utilization`, `attribution`.

## Filters

Each filter has `id`, `type`, `values`, and optionally `mode` (`is` by default, or `starts_with`, `ends_with`, `contains`, `regexp`) and `inverse: true` to exclude the matches.

`metricFilter` limits rows by metric value: a `metric`, an `operator` (`gt`, `lt`, `gte`, `lte`, `b` for between, `nb` for not between, `e`, `ne`) and `values`.

## Top N

A `limit` on a `group` entry selects the top or bottom N values of that dimension: `{"metric": {...}, "sort": "desc", "value": 10}`. It does not cap the number of rows returned.

## Other Options

- `currency`: for example `USD`, `EUR`, `GBP`.
- `includePromotionalCredits`: needs a `timeInterval` of `month`, `quarter` or `year`.
- `secondaryTimeRange` and `displayValues`: compare with an earlier period.

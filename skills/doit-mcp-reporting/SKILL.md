---
name: doit-mcp-reporting
description: Answer cloud cost and reporting questions with the DoiT MCP tools. Use when the user asks about cloud costs, spending, saved reports or Cloud Analytics data, for example "show my costs", "top services", "spending by project", "monthly cost breakdown", "cost by region", "list reports" or "run a report".
---

# DoiT MCP Reporting

Answer report questions through DoiT in the smallest reliable sequence. Prefer a saved report when one already answers the request.

## Reference Files

Read these as needed when building queries:

- `references/dimensions.md` — common dimension IDs and types. Read it when choosing `group`, `dimensions` or `filters`.
- `references/metrics-and-options.md` — metrics, aggregation, time ranges and intervals, data sources, filter options.
- `references/query-examples.md` — ready-to-adapt `run_query` configurations. Read it before building an ad hoc query.
- `templates/report-output.md` — output template for presenting results.

## Tool Order

Use these DoiT MCP tools, skipping steps that are clearly unnecessary:

1. `validate_user` — when the request depends on which account or customer the session is scoped to.
2. `list_reports`, or `list_dimensions` and `get_dimension`.
3. `get_report_results` or `run_query`.

For a plain breakdown by service, project or cloud, or a simple trend, the `cost_breakdown` and `cost_trend` tools (when available) need fewer parameters than `run_query`.

## Saved Report Workflow

Use this when the user names a saved report or asks for something a saved report likely covers.

1. Call `list_reports`, with `name` to filter by a partial name.
2. Match by exact name first, then by obvious close match. If several are plausible, ask one short question.
3. Call `get_report_results` with the report's ID.

A saved report uses its own saved time range and configuration, so its numbers can differ from a fresh query.

## Ad Hoc Query Workflow

Use this when no saved report fits.

1. If a grouping or filter field is unclear, call `list_dimensions`; call `get_dimension` for a dimension's valid values.
2. Build the narrowest `run_query` configuration that answers the question. Start from `references/query-examples.md`.
3. If the result is too broad, add a filter or a `limit` on the group and run it again.

Do not look up dimensions the request already names from `references/dimensions.md`.

## When the Tools Do Not Answer

If a tool fails, returns nothing useful, or the question does not map to a report or query, the `ask_ava_sync` tool (when available) can answer free-form questions about the customer's cloud costs. Include the time range, service and project in the question.

## Output Rules

- Format results with `templates/report-output.md`.
- State the time window, grouping and metric you used, and whether the answer came from a saved report or an ad hoc query.
- For top-N requests, sort and trim before presenting.
- Do not show raw JSON unless the user asks for it.
- If the result is incomplete or ambiguous, say what is missing and ask one focused question.

## Gotchas

- **Dimensions are an ID plus a type**: `{"id": "service_description", "type": "fixed"}`. The type is a separate field, not a prefix on the ID.
- **Metrics are objects**: basic cost is `{"type": "basic", "value": "cost"}`; usage and savings use the values `usage` and `savings`. Use the `metrics` array; `metric` is deprecated.
- **Defaults**: with nothing set, a query returns basic cost for the last 7 days including today, in daily rows, from the billing data source.
- **Relative and custom time ranges**: `timeRange` with mode `last` takes `amount`, `unit` and `includeCurrent`. Explicit dates need `timeRange: {"mode": "custom"}` plus a sibling `customTimeRange` with `from` and `to`.
- **`includeCurrent`**: for one month, `true` means month-to-date and `false` means the previous full month.
- **Filter values match exactly** in the default `is` mode, including case. Cloud providers use IDs: `amazon-web-services`, `google-cloud`, `microsoft-azure`.
- **A group `limit` is not a row cap**: it selects the top or bottom N values of that dimension; time rows still multiply the result.
- **Label and tag keys vary by customer**: discover them with `list_dimensions` instead of guessing.
- **Data source changes the numbers**: switch from `billing` only when the user asks for BigQuery Lens, Kubernetes, DataHub or attribution data.
- **Empty results are valid**: a filter may be too narrow, or the dimension may not apply to the platform.

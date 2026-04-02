---
name: doit-mcp-reporting
description: Use this skill ALWAYS when the user asks about cloud costs, spending, reports, services, invoices, or any DoiT Cloud Analytics data. Triggers on requests like "create a report", "show my costs", "top services", "spending by project", "list reports", "monthly cost breakdown", "cost by region", "how much am I spending", or any cloud cost analysis. This skill MUST be used before calling any DoiT MCP tools directly.
---

# DoiT MCP Reporting

Generate report answers through DoiT in the smallest reliable sequence. Prefer existing reports when they already answer the request.

## Tool Selection

Before starting, check if the DCI CLI is installed by running `command -v dci`.

- **If `dci` is available**: use CLI commands as the primary tool. They are listed in the DCI CLI Command Mapping section below.
- **If `dci` is not available**: you MUST ask the user if they want to install it before proceeding. Tell them: "The DCI CLI is not installed. It's the recommended way to interact with DoiT. Would you like to install it?". Wait for the user's response. If the user agrees, invoke the `doit-mcp-setup` skill to install and authenticate the DCI CLI, then resume this workflow. Only fall back to MCP tools if the user explicitly declines.

Do not mix CLI and MCP calls within the same workflow — pick one and stay consistent.

### Ava Fallback (Ask Ava)

If a DCI CLI command or MCP tool call **fails**, **returns an error**, **returns empty/unexpected results**, or **does not answer the user's question**, fall back to Ava — DoiT's AI assistant that can answer any cloud cost question.

**When to use Ava fallback:**
- A CLI command returns an error or unexpected output
- An MCP tool call fails or times out
- Query results are empty and you cannot determine why
- The user's question is broad, ambiguous, or does not map cleanly to a specific CLI command or MCP tool
- You need contextual or explanatory information that structured tools cannot provide

**How to call Ava:**
```bash
dci ask-ava-sync ephemeral: true, question: "<rephrase the user's question or describe what you need>" --output json
```

- Always set `ephemeral: true` to avoid persisting throwaway conversations.
- Rephrase the question to include relevant context (e.g., time range, service, project) extracted from earlier steps.
- Parse the `answer` field from the JSON response and present it to the user.
- Ava fallback does NOT require `dci` to be the primary tool — use it as a last resort regardless of whether you are in CLI or MCP mode, as long as `dci` is installed.

### DCI CLI Command Mapping

| Operation | DCI CLI Command | MCP Tool (fallback) |
|-----------|----------------|---------------------|
| Validate session | `dci status` | `validate_user` |
| List reports | `dci list-reports --output json` | `list_reports` |
| Get report details | `dci get-report <report-id> --output json` | `get_report_results` |
| Get report config | `dci get-report-config <report-id> --output json` | N/A |
| List dimensions | `dci list-dimensions --output json` | `list_dimensions` |
| Run structured query | `dci query --output json < query.json` | `run_query` |
| Run SQL query | `dci query body.query:"SELECT ..." --output json` | N/A |
| Ask Ava (fallback) | `dci ask-ava-sync ephemeral: true, question: "..." --output json` | N/A |

Use `--output json` for agent parsing, `--output table` for user display. For full DCI CLI query patterns and examples, read `references/dci-cli.md`.

## Reference Files

Read these as needed when building queries:

- `references/dimensions.md` — all dimension IDs (datetime, fixed, label, kubernetes, attribution). Read this when choosing `rows`, `cols`, or `filters` for a query.
- `references/metrics-and-options.md` — metric types, aggregators, time intervals, data sources, filter operators. Read this when configuring query options.
- `references/query-examples.md` — common query patterns with ready-to-use field combinations. Read this before building an ad hoc query to see if a pattern already fits.
- `templates/report-output.md` — output template for presenting results. Copy and fill in when formatting the final report.
- `references/dci-cli.md` — DCI CLI command reference, query modes, and output formats. Read this when using the CLI path.

## Core Tool Order

Use these DoiT MCP tools in this order when applicable:

1. `validate_user`
2. `list_reports` or `list_dimensions`
3. `get_report_results` or `run_query`

Skip only the steps that are clearly unnecessary.

## Session Start

- Call `validate_user` first when the request depends on account access, tenant identity, or you need to confirm the MCP session is scoped correctly.
- If the user already provided enough context and the session is clearly valid, you may skip `validate_user`.

## Existing Report Workflow

Use this when the user mentions a saved report, dashboard-style output, or a named report such as monthly cost overview.

1. Call `list_reports`.
2. Match by exact name first, then by obvious close match.
3. If multiple reports are plausible, ask one short clarifying question instead of choosing arbitrarily.
4. Call `get_report_results` only after you identify the target report.

Prefer this path over `run_query` when a saved report already exists.

## Ad Hoc Query Workflow

Use this when the user wants a slice that is unlikely to map to a saved report.

1. Call `list_dimensions` if the requested grouping or filter fields are unclear.
2. Build the narrowest possible query — see `references/query-examples.md` for starting patterns.
3. Call `run_query`.
4. If the first query is too broad, refine and rerun instead of returning a noisy dump.

Do not call `list_dimensions` for dimensions you already know from the request.

## Output Rules

- Format results using the template in `templates/report-output.md`.
- Include the time window, grouping, and metric assumptions you used.
- Call out when the answer comes from a saved report versus an ad hoc query.
- If the user asks for top-N output, sort and trim before presenting it.
- If the MCP result is incomplete or ambiguous, say what is missing and ask one focused follow-up.

## Gotchas

- **Saved reports first**: Always check `list_reports` before building an ad hoc query. Many common requests already have saved reports — running a fresh query wastes time and may produce slightly different numbers due to config differences.
- **Dimension IDs are prefixed**: Use `fixed:service_description`, not `service_description`. The prefix is required. See `references/dimensions.md` for the full list.
- **Label dimensions are base64-encoded**: Custom labels use the format `label:{base64_key}`. Call `list_dimensions` to discover the actual encoded IDs rather than guessing.
- **Data source matters**: The default is `billing`. Switching to `bqlens`, `kubernetes-utilization`, or `billing-datahub` changes the available dimensions and the numbers. Only switch when the user specifically asks for BQ Lens, K8s, or DataHub data.
- **Metric 0 is cost**: The metric field is a numeric enum, not a string. `0` = cost, `1` = usage, `2` = savings. Using the wrong number silently returns different data.
- **Time interval vs. time range**: `timeInterval` controls the column granularity (day, month, etc.). The time *range* (last 30 days, last quarter) is a separate field. Setting daily interval on a year-long range produces a massive result set.
- **Credits can change totals**: `includeCredits` defaults may vary. If the user's numbers don't match expectations, check whether credits are included or excluded.
- **Currency context**: Results are in the customer's configured currency. If comparing across customers or quoting absolute numbers, note the currency.
- **Empty results are valid**: A query that returns zero rows may mean the filter is too narrow or the dimension doesn't apply to the selected platform — not that something is broken.

## Anti-Patterns

- Do not call `run_query` before deciding whether a saved report already exists.
- Do not enumerate every available report when the user asked for one report.
- Do not use anomaly tools for generic reporting unless the user explicitly asked about anomalies.
- Do not expose raw JSON unless the user asked for raw output.
- Do not build queries with more than 3 row dimensions — the result set becomes unreadable. Narrow with filters instead.
- Do not guess dimension IDs for labels or attributions — always call `list_dimensions` to discover them.

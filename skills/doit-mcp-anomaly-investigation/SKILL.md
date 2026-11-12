---
name: doit-mcp-anomaly-investigation
description: Investigate DoiT cost anomalies with the DoiT MCP tools. Use when the user asks about cost anomalies, spending spikes, unexpected charges or a cost alert, for example "show anomalies", "why did my costs spike", "investigate this charge" or "what happened to my spending", or when an anomaly ID is given.
---

# DoiT MCP Anomaly Investigation

Start from the DoiT anomaly record, then use cost queries to confirm what drove it. Every step is read-only.

## Reference Files

Read these as needed:

- `references/anomaly-fields.md` — anomaly fields, platform values, severity and feedback reasons. Read it when interpreting an anomaly response.
- `references/service-abbreviations.md` — DoiT service names next to the cloud providers' full names. Read it when a service filter returns nothing.
- `references/common-patterns.md` — common root causes by service and anomaly type. Use it to form a hypothesis, then confirm with data.

## Investigation Order

1. Get the anomaly: `get_anomalies` for recent anomalies, or `get_anomaly` when the ID is known.
2. Extract the platform, billing account, project or account, service, SKU, time window and cost impact. The top SKUs' `resourceId` and `operation` are the best narrowing fields.
3. If the `get_anomaly_explanation` tool is available, call it with the anomaly ID. It returns deterministic facts and, separately, an AI-generated likely cause; report the two separately.
4. Confirm the driver with `run_query`: filter to the anomaly's platform, project and service, group by SKU or resource, and use daily rows over a window that starts a week or two before the anomaly so the baseline is visible. See the `doit-mcp-reporting` skill for the query shape.
5. Summarize the confirmed cause, the evidence and next actions.

If the anomaly record already answers the question, stop there.

## When the Tools Do Not Answer

If a tool fails, returns nothing useful, or the question is broader than the anomaly tools cover, the `ask_ava_sync` tool (when available) can answer free-form questions about the customer's cloud costs. Include the anomaly ID, platform, service, time window and cost impact in the question.

The DoiT MCP server reads billing and anomaly data. It does not inspect resources inside the customer's cloud accounts. When the cause needs resource-level evidence (instance lists, logs, configuration), say what to check and where, using `references/common-patterns.md`, and leave the inspection to the user or to cloud tools they have connected.

## Output Rules

- Tie every conclusion to data from a tool result.
- Separate confirmed cause, likely cause and open questions.
- Include the anomaly ID, platform, service, time window and cost impact in every summary.
- Recommend actions only after identifying the cost driver.
- If a missing permission blocked a step, say so and stop.

## Gotchas

- **Service names differ**: DoiT uses short names such as "Amazon EC2"; provider consoles and APIs use full names. Check `references/service-abbreviations.md`.
- **Use the `platform` field** to tell clouds apart; service names alone can be ambiguous.
- **Top SKUs may be empty**: fall back to a service-level query.
- **"No anomalies" may be a threshold effect**: detection sensitivity is configurable per customer.
- **Past feedback matters**: repeated `FAULTY_ANOMALY_DETECTION_MODEL` feedback on similar anomalies suggests false positives for that service.
- **Do not change anything**: acknowledging or updating an anomaly is a separate action the user must ask for.

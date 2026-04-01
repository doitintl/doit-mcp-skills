# DCI CLI Reference for Reporting

## Command Reference

### List Reports

```bash
dci list-reports --output json
```

### Get Report Details

```bash
dci get-report <report-id> --output json
dci get-report-config <report-id> --output json
```

### List Dimensions

```bash
dci list-dimensions --output json
```

### Run a Query

Two modes:

**Inline SQL shorthand** — for quick exploration:

```bash
dci query body.query:"SELECT service_description, SUM(cost) AS total_cost FROM <billing-table> GROUP BY 1 ORDER BY 2 DESC LIMIT 10" --output json
```

**Stdin JSON** — for structured Cloud Analytics queries with metrics, grouping, time ranges, and filters:

```bash
dci query --output json <<'EOF'
{
  "config": {
    "dataSource": "billing",
    "layout": "table",
    "timeInterval": "month",
    "timeRange": {
      "mode": "last",
      "amount": 1,
      "unit": "month",
      "includeCurrent": true
    },
    "metrics": [
      {
        "type": "basic",
        "value": "cost"
      }
    ],
    "group": [
      {
        "id": "service_description",
        "type": "fixed",
        "limit": {
          "metric": {
            "type": "basic",
            "value": "cost"
          },
          "sort": "desc",
          "value": 10
        }
      }
    ]
  }
}
EOF
```

### Output Formats

- `--output json` — for agent parsing (recommended)
- `--output table` — for user display (default)
- `--output yaml` — for YAML output
- `--table-mode wrap` — wrap long cell values
- `--table-columns id,name,amount` — show only specific columns

### Customer Context

Temporary override (preferred):

```bash
DCI_CUSTOMER_CONTEXT=<customer-context> dci list-reports --output json
```

Persistent (only when user explicitly requests):

```bash
dci customer-context set <customer-context>
```

## Query Mode Selection

- Use **SQL shorthand** when the user explicitly asks for SQL or wants a quick billing-table check
- Use **stdin JSON** for structured cost reports, dashboards, or analytics workflows
- Always prefer `--output json` for agent processing

# DCI CLI Reference for Anomaly Investigation

## Command Reference

### List Anomalies

```bash
dci list-anomalies --output json
```

### Get Specific Anomaly

```bash
dci get-anomaly <anomaly-id> --output json
```

### Validate Session

```bash
dci status
```

### List Cloud Incidents

```bash
dci list-known-issues --output json
dci get-known-issue <issue-id> --output json
```

### Follow-Up Queries

After extracting anomaly context, use `dci query` to drill into costs:

```bash
dci query --output json <<'EOF'
{
  "config": {
    "dataSource": "billing",
    "timeInterval": "day",
    "timeRange": {
      "mode": "custom",
      "from": "2026-03-20",
      "to": "2026-03-21"
    },
    "metrics": [
      {
        "type": "basic",
        "value": "cost"
      }
    ],
    "group": [
      {
        "id": "sku_description",
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
    ],
    "filters": [
      {
        "id": "service_description",
        "type": "fixed",
        "inverse": false,
        "values": ["<service-from-anomaly>"]
      }
    ]
  }
}
EOF
```

### Output Formats

- `--output json` — for agent parsing (recommended)
- `--output table` — for user display (default)

### Customer Context

Temporary override (preferred):

```bash
DCI_CUSTOMER_CONTEXT=<customer-context> dci list-anomalies --output json
```

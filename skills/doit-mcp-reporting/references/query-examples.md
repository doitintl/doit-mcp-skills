# Common Query Patterns

Starting points for `run_query`. Adapt the dimensions, filters and time range to the request.

## Top Services Last Month

```json
{
  "config": {
    "dataSource": "billing",
    "metrics": [{ "type": "basic", "value": "cost" }],
    "timeRange": { "mode": "last", "amount": 1, "unit": "month", "includeCurrent": false },
    "timeInterval": "month",
    "group": [
      {
        "id": "service_description",
        "type": "fixed",
        "limit": { "metric": { "type": "basic", "value": "cost" }, "sort": "desc", "value": 10 }
      }
    ]
  }
}
```

## Monthly Cost by Service, Last 6 Months

```json
{
  "config": {
    "metrics": [{ "type": "basic", "value": "cost" }],
    "timeRange": { "mode": "last", "amount": 6, "unit": "month", "includeCurrent": true },
    "timeInterval": "month",
    "group": [{ "id": "service_description", "type": "fixed" }]
  }
}
```

## Daily Cost by Cloud Provider, Last 30 Days

```json
{
  "config": {
    "metrics": [{ "type": "basic", "value": "cost" }],
    "timeRange": { "mode": "last", "amount": 30, "unit": "day", "includeCurrent": true },
    "timeInterval": "day",
    "group": [{ "id": "cloud_provider", "type": "fixed" }]
  }
}
```

## Top SKUs for One Service

```json
{
  "config": {
    "metrics": [{ "type": "basic", "value": "cost" }],
    "timeRange": { "mode": "last", "amount": 1, "unit": "month", "includeCurrent": true },
    "timeInterval": "month",
    "filters": [{ "id": "service_description", "type": "fixed", "values": ["Compute Engine"] }],
    "group": [
      {
        "id": "sku_description",
        "type": "fixed",
        "limit": { "metric": { "type": "basic", "value": "cost" }, "sort": "desc", "value": 10 }
      }
    ]
  }
}
```

## Cost by Project and Region for One Cloud

```json
{
  "config": {
    "metrics": [{ "type": "basic", "value": "cost" }],
    "timeRange": { "mode": "last", "amount": 1, "unit": "month", "includeCurrent": false },
    "timeInterval": "month",
    "filters": [{ "id": "cloud_provider", "type": "fixed", "values": ["google-cloud"] }],
    "group": [
      { "id": "project_id", "type": "fixed" },
      { "id": "region", "type": "fixed" }
    ]
  }
}
```

## Explicit Dates

```json
{
  "config": {
    "metrics": [{ "type": "basic", "value": "cost" }],
    "timeRange": { "mode": "custom" },
    "customTimeRange": { "from": "2026-01-01T00:00:00Z", "to": "2026-01-31T00:00:00Z" },
    "timeInterval": "day",
    "group": [{ "id": "service_description", "type": "fixed" }]
  }
}
```

## Excluding Values

Add `"inverse": true` to a filter to exclude its values:

```json
{ "id": "service_description", "type": "fixed", "values": ["Support"], "inverse": true }
```

## Query Building Tips

1. Start with one group, one time interval and basic cost.
2. Add filters before adding more groups.
3. Use a group `limit` for top-N questions.
4. If the result is still too large, add a `metricFilter`, for example cost greater than 100.
5. Keep `dataSource` at `billing` unless the user asks for BigQuery Lens, Kubernetes, DataHub or attribution data.

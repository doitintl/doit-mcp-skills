# Anomaly Field Reference

Fields commonly returned by `get_anomaly` and `get_anomalies`. Field names can vary between API versions; rely on what the tool result contains.

## Core Fields

| Field | Type | Notes |
|-------|------|-------|
| `id` / `alertId` | string | Unique anomaly identifier |
| `customerId` | string | DoiT customer context |
| `platform` | string | See Platform Values below |
| `billingAccountId` | string | Scoping key for all queries |
| `serviceName` | string | May differ from cloud-native names — see `service-abbreviations.md` |
| `skuName` | string | SKU-level detail when available |
| `costOfAnomaly` | number | Total anomalous cost (USD) |
| `severity` / `severityLevel` | number | Integer — higher = more severe |
| `status` | string | `"ACTIVE"` or `"INACTIVE"` |
| `type` | string | `"COST"` or `"USAGE"` |
| `frequency` | string | `"HOURLY"`, `"DAILY"`, or `"WEEKLY"` |
| `startTime` | Timestamp | When the anomaly began |
| `timestamp` | string | ISO 8601 detection time |
| `value` | number | Observed value |
| `excess` | number | Difference above expected baseline |
| `source` | string | `"real-time"` or `"report"` |

## Platform Values

The `platform` field identifies the cloud or vendor:

| Value | Platform |
|-------|------------|
| `google-cloud` | GCP |
| `amazon-web-services` | AWS |
| `microsoft-azure` | Azure |
| `snowflake` | Snowflake |
| `datadog` | Datadog |
| `databricks` | Databricks |
| `openai` | OpenAI |

## Top SKUs Breakdown

`topSkus` / `top3SKUs` is an array of:

```
{
  cost: number,
  resourceId: string,
  skuDescription: string,
  operation?: string       // e.g., "RunInstances", "CreateVolume", "CreateNatGateway"
}
```

Use `resourceId` and `operation` to narrow follow-up cost queries.

## Sensitivity Levels

| Value | Meaning |
|-------|---------|
| `-1` | Low — fewer alerts, only large spikes |
| `0` | Moderate (default) |
| `1` | High — more alerts, smaller spikes flagged |

## Customer Feedback Reasons

If an anomaly has `customerFeedback`, these are the possible reasons:

**Confirmed anomalous:** `SECURITY_BREACH`, `MISCONFIGURATION`, `UNINTENTIONAL_PROVISIONING`, `EXPECTED_COST_SPIKE`, `ANOMALOUS_OTHER`

**Marked non-anomalous:** `FAULTY_ANOMALY_DETECTION_MODEL`, `INCORRECT_DATA`, `LOW_IMPACT`, `NON_ANOMALOUS_OTHER`

Use feedback to understand if similar past anomalies were false positives.

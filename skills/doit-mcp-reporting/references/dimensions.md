# Dimension Reference

Dimensions used by `run_query` in `group` (rows), `dimensions` (columns) and `filters`. Each is an object with an `id` and a `type`. `list_dimensions` lists what is available for the customer, and `get_dimension` returns a dimension's valid values.

## Fixed Dimensions (type `fixed`)

| ID | Description |
|----|-------------|
| `cloud_provider` | Cloud or platform. Values are IDs: `amazon-web-services`, `google-cloud`, `microsoft-azure` |
| `service_description` | Service name |
| `sku_description` | SKU name |
| `project_id` | GCP project, AWS account or Azure subscription |
| `project_name` | Project display name |
| `billing_account_id` | Billing account |
| `region` | Cloud region |
| `operation` | Operation type |
| `resource_id` | Resource identifier |
| `cost_type` | Cost type |

Other fixed dimensions exist and vary by customer and platform. Confirm with `list_dimensions` (filter `type:fixed`) before using an ID that is not in this table.

## Time (type `datetime`)

Time granularity is normally set with `timeInterval` (`hour`, `day`, `week`, `month`, `quarter`, `year`), which adds the time columns for you. Datetime dimensions such as `year`, `month` and `day` can also be placed in `dimensions` explicitly.

## Labels, Tags and Other Types

| Type | Description |
|------|-------------|
| `label` | Resource labels |
| `tag` | Resource tags |
| `project_label` | Project-level labels |
| `system_label` | System labels |
| `attribution`, `attribution_group` | Attributions and attribution groups |
| `allocation`, `allocation_rule` | Allocations and their rules |
| `gke`, `gke_label` | GKE fields and labels |

The IDs for these types are specific to each customer. Get them from `list_dimensions`, for example with filter `type:label`, and do not guess them.

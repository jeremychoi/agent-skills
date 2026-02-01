---
name: superset-api
description: "Interact with Apache Superset REST API to manage dashboards, charts, datasets, databases, and execute SQL queries. Use when users need to programmatically work with Superset for: (1) Listing or managing dashboards and charts, (2) Creating or querying datasets, (3) Executing SQL queries via SQL Lab, (4) Managing database connections, (5) Retrieving chart data with filters and metrics, (6) Generating guest tokens for embedded dashboards. Requires a Superset host URL."
---

# Apache Superset API

Programmatic access to Apache Superset business intelligence features via REST API.

## Configuration

- **Host URL**: Base URL of the Superset instance (e.g., `http://localhost:8088`)
- **Swagger UI**: Full API documentation at `http://<host>/swagger/v1`

## Authentication Flow

1. **Login** → `POST /api/v1/security/login` with username/password to get `access_token`
2. **Get CSRF** → `GET /api/v1/security/csrf_token/` (required for mutating operations)
3. **Use tokens** → Include `Authorization: Bearer <access_token>` on all requests; add `X-CSRFToken: <csrf_token>` header for POST/PUT/DELETE

**Important**: Use a cookie jar (`-c` and `-b` flags with curl) to maintain session cookies across requests. The CSRF token requires an active session to work properly.

## Key Concepts

- **CSRF Token**: Required for all mutating operations (POST, PUT, DELETE)
- **Pagination**: Most list endpoints support `page` and `page_size` query parameters
- **Filtering**: Use `q` parameter with Rison-encoded filter syntax

## Core Capabilities

| Capability | Key Endpoints |
|------------|---------------|
| Dashboards | `/api/v1/dashboard/` - CRUD, export/import |
| Charts | `/api/v1/chart/` - CRUD; `/api/v1/chart/data` - execute queries |
| Datasets | `/api/v1/dataset/` - create from tables or SQL |
| Databases | `/api/v1/database/` - manage connections |
| SQL Lab | `/api/v1/sqllab/execute/` - run SQL queries (sync/async) |
| Security | `/api/v1/security/guest_token/` - embedded dashboard tokens with RLS |

## Chart Types Reference

| Chart Type (`viz_type`) | Description |
|-------------------------|-------------|
| `echarts_area` | Area under lines to show ratios, proportions, or percentages over a dimension (e.g., time) |
| `echarts_timeseries_bar` | Bars to visualize how metrics change over a dimension (e.g., time) |
| `dist_bar` | Bar chart (legacy) - easy-to-understand categorical data visualization |
| `big_number_total` | Emphasize a single metric or KPI at a specific point in time |
| `big_number` | Big number with trendline showing recent metric changes |
| `funnel` | Funnel chart for pipeline stages visualization |
| `gauge_chart` | Gauge showing partial progress towards a specific number |
| `graph_chart` | Nodes/circles with edges to visualize connected objects or events |
| `echarts_timeseries_line` | Lines showing how metrics change over a dimension (e.g., time) |
| `mixed_timeseries` | Multiple visualizations in a single chart with shared X-axis |
| `pie` | Circular chart showing proportions as slices |
| `pivot_table_v2` | Aggregated table with grouped values across discrete categories |
| `radar` | Visualize parallel metrics across multiple groups or categories |
| `sankey` | Flow diagram showing relative metric size via flow line thickness |
| `echarts_timeseries_scatter` | Dots showing how metrics change over a dimension |
| `smooth` | Smoothed line chart connecting data points |
| `stepped_line` | Line chart using 90-degree stepped lines |
| `table` | Classic spreadsheet-like view of data or aggregated metrics |
| `tree_chart` | Excel at visualizing hierarchy |
| `treemap` | Scaled rectangles showing same metric across different groups |

## Adding Charts to Dashboards

When creating a dashboard with charts via API, follow this sequence:

1. **Create the chart** → `POST /api/v1/chart/` with `datasource_id`, `viz_type`, `params`
2. **Create the dashboard** → `POST /api/v1/dashboard/` with `dashboard_title`
3. **Link chart to dashboard** → `PUT /api/v1/chart/{chart_id}` with `{"dashboards": [dashboard_id]}`
4. **Set dashboard layout** → `PUT /api/v1/dashboard/{dashboard_id}` with `position_json`

**Critical**: Step 3 is required. The `position_json` defines layout but does NOT link the chart to the dashboard. You must explicitly update the chart's `dashboards` array to associate it with the dashboard, otherwise the chart will not render.

Example `position_json` structure:
```json
{
  "DASHBOARD_VERSION_KEY": "v2",
  "ROOT_ID": {"type": "ROOT", "id": "ROOT_ID", "children": ["GRID_ID"]},
  "GRID_ID": {"type": "GRID", "id": "GRID_ID", "children": ["ROW-1"], "parents": ["ROOT_ID"]},
  "ROW-1": {"type": "ROW", "id": "ROW-1", "children": ["CHART-1"], "parents": ["ROOT_ID", "GRID_ID"]},
  "CHART-1": {"type": "CHART", "id": "CHART-1", "children": [], "parents": ["ROOT_ID", "GRID_ID", "ROW-1"], "meta": {"chartId": <chart_id>, "width": 12, "height": 50}}
}
```

## Usage Pattern

1. Obtain the Superset host URL from user
2. Fetch Swagger spec from `http://<host>/swagger/v1` to discover available endpoints
3. Authenticate to get access token and CSRF token
4. Make API calls with appropriate headers

Refer to `http://<host>/swagger/v1` for complete endpoint documentation, parameters, and response schemas.

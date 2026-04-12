# Data Pipeline with Tool Integration

Fetch data from an external source, then analyze it with AI.

## INPUTS

- `metric_name` (string): Name of the metric to fetch
- `timeframe` (string: 7d): Time range for data retrieval

## STEP 1: Fetch Metrics

@tool(analytics_server, get_metrics, {"metric": "{{metric_name}}", "period": "{{timeframe}}", "format": "json"})
@output(metrics_data)

## STEP 2: Analyze Trends

Using the metrics data for {{metric_name}} over the last {{timeframe}}, identify:

1. Overall trend direction (increasing, decreasing, stable)
2. Notable anomalies or outliers
3. Seasonal patterns if any
4. Predicted trajectory for the next period

Provide specific numbers and percentages where possible.

# ELK Stack

> Elasticsearch, Logstash, Kibana

## What is the ELK Stack?

The ELK stack is a common setup for ingesting, storing, searching, and visualizing event data.

ELK stands for:

- **Elasticsearch**: stores and searches data
- **Logstash**: ingests and transforms data
- **Kibana**: visualizes and explores data

Together, they form a pipeline:

```text
Data Source -> Logstash -> Elasticsearch -> Kibana
```

## Beats

Beats are lightweight agents that collect data from systems and send it into the Elastic stack.

Common examples:

- `Filebeat`: ships log files
- `Metricbeat`: ships system and service metrics
- `Packetbeat`: captures network traffic metadata
- `Auditbeat`: ships audit/security data
- `Heartbeat`: checks service uptime

In many setups, Beats send data to Logstash for parsing and enrichment, though they can also send data directly to Elasticsearch.

## Role of Each Part

- **Logstash** receives raw data and converts it into structured events
- **Elasticsearch** stores those events and makes them searchable
- **Kibana** lets users explore the data with searches, filters, charts, and dashboards

Each tool has a separate responsibility, which makes the stack easier to scale and debug.

## End-To-End Workflow

1. A source system produces raw data
2. Logstash parses and normalizes that data
3. Elasticsearch indexes the structured event
4. Kibana queries Elasticsearch and displays the results

This usually works best when field names and datatypes are consistent from the start.

## Example

Raw log line:

```text
2026-04-09T20:00:00Z ERROR auth-api user_id=42 response_time_ms=124 message="User login failed"
```

After Logstash parsing:

```json
{
    "@timestamp": "2026-04-09T20:00:00Z",
    "level": "ERROR",
    "service": "auth-api",
    "message": "User login failed",
    "user_id": 42,
    "response_time_ms": 124
}
```

After Elasticsearch indexing, fields might use datatypes such as:

- `@timestamp`: `date`
- `level`: `keyword`
- `service`: `keyword`
- `message`: `text`
- `user_id`: `long`
- `response_time_ms`: `integer`

In Kibana, that same event can be:

- searched in Discover
- filtered by `service: auth-api`
- counted in an error chart
- added to a dashboard

## Why Datatypes Matter

Datatypes connect the whole stack.

Examples:

- a timestamp should be mapped as `date` so Kibana can use time filters
- a field like `level` should usually be `keyword` for exact filtering
- numeric fields should stay numeric so Elasticsearch can calculate averages, ranges, and counts

If datatypes are wrong, search and visualization become harder.

## Common Problems

- sending raw, unparsed logs straight into Elasticsearch
- using inconsistent field names across sources
- storing timestamps as strings instead of `date`
- using `text` where `keyword` is needed
- building dashboards before validating the data in Discover

## Troubleshooting Flow

If something looks wrong in Kibana, work backward:

1. Check whether the document exists in Elasticsearch
2. Check whether the fields and datatypes are correct
3. Check the Logstash output event shape
4. Check the original source data

That is usually the fastest way to find where the problem starts.

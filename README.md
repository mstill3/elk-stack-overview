# ELK Stack

This repo contains beginner-friendly notes for understanding the ELK stack:

- [01-elasticsearch-basics.md](docs/01-elasticsearch.md)
- [02-logstash-basics.md](docs/02-logstash.md)
- [03-kibana-basics.md](docs/03-kibana.md)
- [04-how-elk-works-together.md](docs/04-elk-stack.md)

## What The ELK Stack Is

ELK stands for:

- **Elasticsearch**: stores and searches data
- **Logstash**: ingests and transforms data
- **Kibana**: visualizes and explores data

Together, they form a pipeline for collecting data, storing it, and making it useful.

## The Core Flow

Think of the stack like this:

1. Logstash receives data from files, APIs, message queues, or other systems.
2. Logstash parses and cleans the data so it has a consistent structure.
3. Elasticsearch stores that structured data in indexes.
4. Kibana lets you search, filter, visualize, and build dashboards from the data in Elasticsearch.

## What Each Part Does

Each part of the stack has a different responsibility:

- **Elasticsearch** is best at fast search, filtering, and aggregations across large datasets.
- **Logstash** is best at transforming messy input into consistent records.
- **Kibana** is best at helping people inspect and understand the data.

If you keep those responsibilities separate, the stack is easier to reason about.

## A Simple Example Event

Imagine this raw log line:

```text
2026-04-09T20:00:00Z ERROR auth-api user_id=42 response_time_ms=124 message="User login failed"
```

That line starts as plain text. Logstash can parse it into fields like:

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

Elasticsearch stores that record in an index. Kibana then makes it possible to:

- search for failed login events
- filter only `service: auth-api`
- chart errors over time
- build a dashboard for authentication activity

## Why Clean Structure Matters

The stack works best when your data is structured consistently.

That usually means:

- field names stay consistent across sources
- timestamps are stored as `date`
- counts and identifiers use appropriate numeric datatypes
- exact-match fields use `keyword`
- free-form message content uses `text`

Poor structure usually causes most beginner problems.

## A Good Beginner Workflow

A practical way to learn the stack is:

1. Start with one sample log source.
2. Parse it with Logstash into predictable fields.
3. Verify the field names and datatypes in Elasticsearch.
4. Explore the indexed records in Kibana Discover.
5. Build one small dashboard only after the data looks clean.

## Key Terms

- **Event**: one unit of incoming data in Logstash
- **Document**: one stored JSON record in Elasticsearch
- **Index**: a collection of related documents
- **Mapping**: the datatype definitions for fields in an index
- **Data view**: Kibana's way to point at one or more indexes
- **Dashboard**: a page made of multiple visualizations

## What These Notes Cover

These notes focus on the basics needed to understand the stack well enough to:

- ingest data
- choose sensible datatypes
- search and filter correctly
- avoid common beginner mistakes
- understand how the three tools fit together

## Recommended Reading Order

Read in this order:

1. Elasticsearch basics
2. Logstash basics
3. Kibana basics
4. How they work together

If you are setting up a real pipeline, keep the Elasticsearch and Logstash pages open at the same time. Most practical issues come from the relationship between parsing and datatypes.

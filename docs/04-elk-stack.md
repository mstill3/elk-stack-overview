# ELK Stack

> Elasticsearch, Logstash, Kibana

## What is the ELK Stack?

ELK stands for:

- **Elasticsearch**: stores and searches data
- **Logstash**: ingests and transforms data
- **Kibana**: visualizes and explores data

Together, they form a pipeline for collecting data, storing it, and displaying it.

>>> TODO

## Workflow

Think of the stack like this:

1. Logstash receives data from files, APIs, message queues, or other systems
2. Logstash parses and cleans the data so it has a consistent structure
3. Elasticsearch stores that structured data in indexes
4. Kibana lets you search, filter, visualize, and build dashboards from the data in Elasticsearch

```text
Data_Source -> Logstash -> Elasticsearch -> Kibana

Data_Source -> Ingestion -> Transform/Enrich -> Storage -> UI
```

## Pipeline

The ELK stack is easiest to understand as a pipeline:

```text
Data_Source -> Logstash -> Elasticsearch -> Kibana

Data_Source -> Ingestion -> Transform/Enrich -> Storage -> UI
```

That single line is the core of the whole stack. Everything else is detail around how well each stage does its job.

## Step 1: Start With Raw Input

The source could be:

- application logs
- web server logs
- security events
- metrics converted into events
- messages from Kafka or APIs

At this stage, the data may be raw, inconsistent, or hard to query.

Sources vary widely in quality. Some already produce clean JSON, while others only produce plain text lines that need heavy parsing.

## Step 2: Shape The Event

Logstash receives the raw data and transforms it into structured events.

Typical work done here:

- parse message text
- extract fields
- standardize timestamps
- assign consistent field names
- drop noisy fields
- enrich records with metadata

This is where raw data becomes a structured event with useful fields and better datatypes.

Example output event:

```json
{
    "@timestamp": "2026-04-09T20:00:00Z",
    "service": "auth-api",
    "level": "ERROR",
    "message": "User login failed",
    "user_id": 42,
    "response_time_ms": 124
}
```

If Logstash does this stage well, Elasticsearch and Kibana become much easier to work with.

## Step 3: Store And Index It

Elasticsearch stores the event in an index and makes it searchable.

Once indexed, you can:

- search by text
- filter by exact values
- aggregate counts and metrics
- query by time range

Elasticsearch depends on mappings and datatypes to make those operations work correctly.

## Step 4: Explore And Visualize It

Kibana lets users work with the indexed data through:

- Discover for raw events
- visualizations for charts
- dashboards for monitoring and analysis

Kibana is where people usually interact with the stack day to day.

## Follow One Event End To End

It helps to track one event from start to finish.

### Raw input

```text
2026-04-09T20:00:00Z ERROR auth-api user_id=42 response_time_ms=124 message="User login failed"
```

### After Logstash parsing

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

### After Elasticsearch indexing

The event is stored as a document in an index such as `application-logs`, with field datatypes like:

- `@timestamp`: `date`
- `level`: `keyword`
- `service`: `keyword`
- `message`: `text`
- `user_id`: `long`
- `response_time_ms`: `integer`

### In Kibana

That same document can now be:

- searched in Discover
- filtered by `service: auth-api`
- counted in an error-rate chart
- included in an operations dashboard

## One Realistic Walkthrough

Imagine an application writes this raw log line:

```text
2026-04-09T20:00:00Z ERROR auth-api user_id=42 response_time_ms=124 message="User login failed"
```

Logstash parses it into fields:

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

Elasticsearch stores it in an index such as `application-logs`.

Kibana can then show:

- a chart of login failures over time
- a table of recent failed logins
- a dashboard filtered to `service: auth-api`

## Why Field Types Tie The Stack Together

Datatypes are one of the clearest links between all three tools.

Examples:

- Logstash extracts a timestamp string
- Elasticsearch maps that field as `date`
- Kibana uses that `date` field for time filtering and date histograms

Another example:

- Logstash extracts `level` as `ERROR`
- Elasticsearch maps `level` as `keyword`
- Kibana uses it in exact filters and top-values charts

If the datatype is wrong at the Elasticsearch stage, Kibana often becomes much less useful.

## Why This Separation Helps

Each part has a clear responsibility:

- **Logstash** handles ingestion and transformation
- **Elasticsearch** handles storage and search
- **Kibana** handles exploration and visualization

That separation makes the platform flexible and scalable.

It also makes troubleshooting easier because you can ask:

- did the source send the event
- did Logstash parse it correctly
- did Elasticsearch map and store it correctly
- does Kibana show it correctly

## Common Beginner Mistakes

- sending messy data into Elasticsearch without parsing it first
- using inconsistent field names across sources
- storing timestamps as plain strings instead of `date`
- using `text` when `keyword` is needed for filtering
- trying to build dashboards before validating data in Discover

## How To Trace A Problem Through The Stack

When something looks wrong in Kibana, work backward:

1. Check whether the document exists in Elasticsearch
2. Check whether the fields and datatypes are correct
3. Check the Logstash output event shape
4. Check whether the original source data contains the expected values

This method is usually faster than guessing inside Kibana first.

## Practical Rules Of Thumb

- parse as early as possible
- choose datatypes deliberately
- keep field names consistent across sources
- use Kibana Discover before building dashboards
- test with a small sample before sending high-volume data

## If You Get Lost, Reduce It To Four Questions

If ELK feels confusing at first, simplify it to these questions:

1. What is the raw input?
2. What fields should exist after parsing?
3. What datatype should each field have?
4. What question do I want Kibana to answer?

Those four questions usually expose where the real issue is.

## A Good Learning Order

If you are new to the stack, focus on this order:

1. Understand what a document, field, index, and mapping are in Elasticsearch
2. Learn how Logstash converts raw input into structured events
3. Use Kibana Discover to confirm the indexed data looks correct
4. Build visualizations only after the field structure is clean

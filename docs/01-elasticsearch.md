# Elasticsearch

> Distributed Search & Analytics Engine

## What is Elasticsearch?

Elasticsearch is a distributed search and analytics engine. It is designed to store large volumes of structured data and make that data searchable very quickly.

| Name                            | Port | Protocol |
| ------------------------------- | ---- | -------- |
| Default HTTP API                | 9200 | TCP      |
| Default transport between nodes | 9300 | TCP      |

It is commonly used for:

- log storage
- full-text search
- metrics and observability
- security analytics
- application search

It is especially powerful when your data has clear field types and you need to search, filter, and aggregate in near real time.

## Keywords

Elasticsearch terminology:

- **Document**: a record
- **Index**: collection of records
- **Field**: a property on a record
- **Mapping**: defines the datatype for each field
- **Node**: a single running instance of Elasticsearch on a given server
- **Cluster**: collection of nodes that work together as a single Elasticsearch system; maintains a shared state that tracks indexes, shards, and node assignments
- **Shard**: a partition of an index that allows data to be distributed across nodes
  - **Primary shard**: the original shard where data is written
  - **Replica shard**: a copy of a primary shard used for redundancy and read scaling

## Key Concepts

### Document

A document is a JSON object stored in Elasticsearch.

Example:

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

Each document should represent one logical event or entity. In log pipelines, one log event usually becomes one document.

### Index

An index is a collection of related documents.

For more information, see [How To Organize Indexes](#how-to-organize-indexes).

Examples:

- `application-logs`
- `security-events`
- `api-metrics`

Indexes are often organized by data type, source system, or time period.

### Inverted Index

Elasticsearch uses an inverted index under the hood.

Instead of storing documents directly for lookup, it builds a structure mapping terms to documents.

Each term is stored along with the list of documents that contain it.

Example:

"login failed" → doc1, doc7, doc42

This is what makes full-text search extremely fast.

### Field

A field is a key inside a document.

In the example above, these are fields:

- `@timestamp`
- `service`
- `level`
- `message`
- `user_id`
- `response_time_ms`

Fields are what you search, filter, sort, and aggregate on.

### Mapping

A mapping defines the datatype of each field in an index.

Common datatypes:

- `text`: analyzed strings for full-text search
- `keyword`: exact values for filtering and aggregation
- `date`: timestamps and dates
- `long` or `integer`: whole numbers
- `float` or `double`: decimal numbers
- `boolean`: true or false

For more information, see [Datatypes](#datatypes).

Example mapping:

```json
{
    "mappings": {
        "properties": {
            "@timestamp": {
                "type": "date"
            },
            "service": {
                "type": "keyword"
            },
            "level": {
                "type": "keyword"
            },
            "message": {
                "type": "text"
            },
            "user_id": {
                "type": "long"
            },
            "response_time_ms": {
                "type": "integer"
            }
        }
    }
}
```

Mappings do not need to be explicitly defined, as Elasticsearch can infer them from the indexed logs; however, relying on this behavior is **NOT** recommended in production environments.

For more information, see [Dynamic Mapping](#dynamic-mapping).

Mappings are important because Elasticsearch needs to know how each field should behave.

For example:

- `message` as `text` supports full-text search
- `level` as `keyword` supports exact filtering
- `@timestamp` as `date` supports time-range queries
- `user_id` as `long` supports numeric aggregations
- `response_time_ms` as `integer` supports averages and ranges

## Datatypes

### `text`

Use `text` for human-readable content you want to search by words or phrases.

Examples:

- log messages
- descriptions
- free-form event text

### `keyword`

Use `keyword` for exact values.

Examples:

- service names
- log levels
- hostnames
- environment names
- IDs stored as strings

If you want exact filtering, sorting, or aggregations, `keyword` is usually the right choice.

#### `text` vs `keyword`

Beginners often confuse these.

- Use `text` when you want analyzed full-text search
- Use `keyword` when you want exact matches

Example:

- `message: "User login failed"` should usually be `text`
- `level: "ERROR"` should usually be `keyword`

Some fields may have both forms available, but the key idea is still the same: full-text search and exact matching are different needs.

### Numeric

Use numeric datatypes for values you want to compare or aggregate.

Common choices:

- `integer`
- `long`
- `float`
- `double`

Examples:

- response time
- byte count
- user ID when it is numeric
- status code

### `date`

Use `date` for timestamps and dates. This is required for time-based analysis.

### `boolean`

Use `boolean` for true/false values.

Examples:

- `success`
- `authenticated`
- `is_internal`

## Why Datatypes Matter

Datatypes affect how Elasticsearch stores and queries data.

If the wrong datatype is chosen, searches and dashboards can behave incorrectly.

Common failure patterns:

- trying to build a date histogram on a string field
- trying to aggregate on a field that was mapped incorrectly
- filtering exact values on a field that was intended for full-text search
- mixing numeric and text representations of the same field across datasets

## What is Data Indexing?

When a document is sent to Elasticsearch, it is indexed.

Indexing means Elasticsearch stores the document and builds internal data structures that make search and aggregation fast.

During indexing, Elasticsearch applies the mapping for the target index. If the incoming data does not fit the expected datatype, the document may be rejected or the field may behave in an unexpected way.

That is why datatype planning matters before large amounts of data are ingested.

## Dynamic Mapping

Elasticsearch can infer field datatypes automatically when new fields appear.

This is convenient for experimentation, but it can cause problems in real pipelines.

Dynamic mapping can silently introduce bad data models that are difficult to fix later, especially once indexes are large

Examples of issues:

- a numeric field gets stored as text because the first value looked like a string
- a timestamp stays a plain string instead of a `date`
- the same field name ends up with inconsistent meaning across sources

For learning and quick testing, dynamic mapping is useful. For stable pipelines, explicit mappings are safer.

## Common Operations

### Index a document

```json
POST /application-logs/_doc
{
    "@timestamp": "2026-04-09T20:00:00Z",
    "service": "auth-api",
    "level": "ERROR",
    "message": "User login failed",
    "user_id": 42,
    "response_time_ms": 124
}
```

### Search documents

```json
GET /application-logs/_search
{
    "query": {
        "match": {
            "message": "login"
        }
    }
}
```

`match` works well for analyzed text fields such as `message`.

### Filter documents

```json
GET /application-logs/_search
{
    "query": {
        "term": {
            "service": "auth-api"
        }
    }
}
```

`term` is typically used for exact values on fields like `keyword`, numbers, and booleans.

### Aggregate documents

```json
GET /application-logs/_search
{
    "size": 0,
    "aggs": {
        "by_level": {
            "terms": {
                "field": "level"
            }
        }
    }
}
```

Aggregations let you answer questions like:

- how many errors occurred
- which services produced the most logs
- what the average response time is

## Query Types

- `match`: useful for full-text search
- `term`: useful for exact values
- `range`: useful for dates and numeric values
- `bool`: combines multiple conditions

Example `range` query:

```json
GET /application-logs/_search
{
   "query": {
        "range": {
            "@timestamp": {
                "gte": "now-15m",
                "lte": "now"
            }
        }
    }
}
```

## How To Organize Indexes

A simple and common pattern is to group similar data into separate indexes.

Examples:

- `application-logs`
- `audit-logs`
- `network-events`

Good index organization helps with:

- permissions
- retention
- easier debugging
- cleaner Kibana data views

## How Elasticsearch Scales

Elasticsearch distributes data across nodes.

- **Primary shards** split data into pieces.
- **Replicas** are copies of primary shards for fault tolerance and read scaling.

This is one reason Elasticsearch can scale horizontally.

Elasticsearch is designed to distribute and duplicate data across nodes.

## Elasticsearch Workflow

1. An application writes a raw log event
2. Logstash parses that event into structured fields
3. Elasticsearch receives the event as JSON
4. The index mapping assigns field types like `date`, `keyword`, and `integer`
5. The document becomes searchable and usable in Kibana

## Good Habits

- Decide field names early and keep them consistent
- Be deliberate about datatypes, especially `date`, `keyword`, and numeric fields
- Keep message bodies separate from exact-match fields
- Validate sample documents before ingesting large volumes of data
- Use small test indexes while learning

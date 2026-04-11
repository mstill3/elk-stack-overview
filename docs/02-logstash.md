# Logstash

> Ingestion & transformation layer

## What is Logstash?

Logstash is a data processing pipeline. It ingests data from one or more sources, transforms that data, and sends it to one or more destinations.

Its main purpose is moving and transforming events between systems. A primary goal is to convert raw data into structured events

It is often used to:

- read logs from files
- ingest data from Kafka, Beats, or APIs
- **parse unstructured text into structured fields**
- enrich events before storing them
- route data to Elasticsearch

### Logstash's Role

Logstash is most useful when incoming data is inconsistent.

Its job is usually to:

- receive raw input
- identify useful pieces of information
- convert those pieces into named fields
- normalize datatypes where possible
- send the cleaned event onward

Logstash is where you shape the data to what Elasticsearch & Kibana expect.

## Keywords

Logstash terminology:

- **Event**: structured record with fields, similar to a JSON document
- **Field**: a property on a record

## Logstash's Pipeline

A Logstash pipeline has 3 stages:

1. **Input**
2. **Filter**
3. **Output**

Basic shape:

```conf
input {}
filter {}
output {}
```

Every event moves through these 3 stages in order.

### 1. Inputs

The input stage defines where events come from.

Examples:

- file input
- beats input
- kafka input
- http input

Example:

```conf
input {
    file {
        path => "/var/log/app.log"
        start_position => "beginning"
    }
}
```

Inputs determine how data **enters** the pipeline.

### 2. Filters

The filter stage parses, transforms, and enriches events.

Common filters:

- `grok`: parse unstructured text with patterns
- `mutate`: rename, remove, or update fields
- `date`: convert strings into timestamps
- `json`: parse JSON text
- `geoip`: enrich IP addresses with location data

This is where most of the real "work" happens.

Example:

```conf
filter {
    grok {
        match => {
            "message" => "%{TIMESTAMP_ISO8601:log_time} %{LOGLEVEL:level} %{NOTSPACE:service} user_id=%{INT:user_id} response_time_ms=%{INT:response_time_ms} message=\"%{GREEDYDATA:message}\""
        }
    }

    date {
        match => [ "log_time", "ISO8601" ]
        target => "@timestamp"
    }

    mutate {
        convert => {
            "user_id" => "integer"
            "response_time_ms" => "integer"
        }
    }
}
```

In that example:

- `grok` extracts named pieces from the raw log line
- `date` converts a string timestamp into a proper event timestamp
- `mutate` converts numeric-looking fields into numeric values

### 3. Outputs

The output stage defines where processed events are **sent**.

Examples:

- Elasticsearch
- stdout
- file
- Kafka

Example:

```conf
output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
        index => "application-logs"
    }
}
```

While Elasticsearch is the most common output in ELK, sending data to `stdout` during development is also useful because it lets you inspect exactly what the event looks like after filtering.

Example:

```conf
output {
    stdout {
        codec => rubydebug
    }
}
```

## Logstash Event

Logstash processes data as events.

A good Logstash pipeline turns messy input into predictable fields and datatypes before the data reaches Elasticsearch.

Example event after parsing:

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

## Common Ways To Parse Data

There are several ways to parse incoming data.

### Parse plain text with `grok`

Use `grok` when logs are plain text but still follow a predictable pattern.

### Parse structured JSON with `json`

Use `json` when the incoming message already contains JSON and just needs to be turned into fields.

### Rename or clean fields with `mutate`

Use `mutate` to:

- rename fields
- remove fields
- add fields
- convert field values

### Normalize timestamps with `date`

Use `date` to make sure time values become `@timestamp`, which Kibana expects for time-based analysis.

>>>> TODO: stopped here

## Why This Step Matters

Raw logs are often inconsistent. Logstash helps by:

- extracting useful fields from free text
- normalizing timestamps
- assigning consistent names to fields
- cleaning bad or unnecessary data
- enriching records before indexing

This is especially important when multiple systems produce logs in different formats.

## Preparing Clean Field Types

You prefer datatypes, so here is the key idea: Logstash should help prepare values so Elasticsearch can map them correctly.

Examples:

- timestamps should be converted into real dates
- numeric strings should be converted when appropriate
- booleans should not remain ambiguous text like `"true"` or `"false"` if you can avoid it

If Logstash leaves values messy, Elasticsearch may infer the wrong mapping.

## A Better Example

Input log line:

```text
2026-04-09T20:00:00Z ERROR auth-api user_id=42 response_time_ms=124 message="User login failed"
```

After parsing, Logstash might produce:

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

A more complete result might look like this:

```json
{
    "@timestamp": "2026-04-09T20:00:00Z",
    "level": "ERROR",
    "service": "auth-api",
    "message": "User login failed",
    "user_id": 42,
    "response_time_ms": 124,
    "environment": "prod"
}
```

## Sample Pipeline

```conf
input {
    file {
        path => "/var/log/app.log"
        start_position => "beginning"
    }
}

filter {
    grok {
        match => {
            "message" => "%{TIMESTAMP_ISO8601:log_time} %{LOGLEVEL:level} %{NOTSPACE:service} user_id=%{INT:user_id} response_time_ms=%{INT:response_time_ms} message=\"%{GREEDYDATA:message}\""
        }
    }

    date {
        match => [ "log_time", "ISO8601" ]
        target => "@timestamp"
    }

    mutate {
        convert => {
            "user_id" => "integer"
            "response_time_ms" => "integer"
        }
        add_field => { "environment" => "prod" }
        remove_field => [ "log_time" ]
    }
}

output {
    elasticsearch {
        hosts => ["http://localhost:9200"]
        index => "application-logs"
    }
}
```

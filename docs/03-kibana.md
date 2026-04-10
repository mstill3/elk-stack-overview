# Kibana

## What Kibana Does

Kibana is the visualization and exploration interface for Elasticsearch data. It lets users search records, build charts, and create dashboards.

It is commonly used for:

- log exploration
- dashboard creation
- operational monitoring
- security investigation
- ad hoc data analysis

Kibana is often the first place where users notice whether the underlying data model is clean or messy.

## How Kibana Fits Into The Stack

Kibana does not store the main dataset itself. It queries Elasticsearch and presents the results in a user-friendly interface.

That means many Kibana problems are really data problems from earlier in the pipeline.

Examples:

- missing fields
- incorrect datatypes
- timestamps stored as text
- inconsistent field names

## Key Parts Of Kibana

### Data Views

A data view tells Kibana which Elasticsearch indexes to use.

Example:

- `application-logs*`

This allows Kibana to search multiple indexes that match a pattern.

If your data is spread across similar indexes, a data view lets Kibana treat them as one logical dataset.

### Discover

Discover is used for exploring raw documents.

You can:

- search text
- filter fields
- inspect individual records
- sort by timestamp

Discover is usually the best starting point when validating ingestion.

When a pipeline is new, Discover should be your first stop before building charts.

### Visualizations

Visualizations turn query results into charts such as:

- bar charts
- line charts
- pie charts
- tables
- metric tiles

Visualizations are built on top of fields and aggregations. If the fields are poorly structured, the charts will be confusing or impossible to build.

### Dashboards

Dashboards combine multiple visualizations into one view.

A dashboard might show:

- log volume over time
- top services by error count
- recent critical events
- event counts by environment

Dashboards are useful when you want a repeatable operational view rather than one-off exploration.

## A Good Beginner Workflow In Kibana

A common workflow looks like this:

1. Create a data view for the target indexes.
2. Open Discover and confirm documents look correct.
3. Verify that `@timestamp` and key fields are usable.
4. Build one or two visualizations.
5. Add those visualizations to a dashboard.

## Time Is Usually The Starting Point

Many ELK use cases depend on time-based data. If your documents have a valid `@timestamp` field with the correct datatype, Kibana can filter and chart events over time.

This is one of the most important reasons to normalize timestamps in Logstash.

Without a correct time field, many of Kibana's most useful features become harder to use.

## Common Ways To Explore Data

Kibana supports:

- free-text search
- field-based filters
- time-range filters
- aggregations through visual tools

Examples:

- search for `login failed`
- filter `service: auth-api`
- show the last 15 minutes

Searching is usually a combination of:

- text search for message content
- exact filters for fields like `service`, `level`, or `environment`
- time filters to narrow the analysis window

## Why Field Types Matter Here

Kibana depends heavily on Elasticsearch datatypes.

Examples:

- a `date` field enables timelines and date histograms
- a `keyword` field works well for exact filters and top-values charts
- numeric fields support averages, sums, and ranges
- a `text` field is useful for reading and searching message content

If a field has the wrong datatype, the visualization options may not make sense.

## Questions Kibana Should Help You Answer

Kibana helps answer questions like:

- how many failed login events happened in the last hour
- which service produced the most errors today
- when did a spike begin
- which services are generating warnings
- what raw events are behind a chart anomaly

## Good First Visualizations

Good first visualizations include:

- events over time
- top log levels
- top services
- recent error table
- average `response_time_ms` by service

These are simple, but they quickly reveal whether your data is being indexed correctly.

## Common Beginner Mistakes

- building dashboards before checking raw documents in Discover
- expecting Kibana to fix poor field design
- using the wrong time range and thinking data is missing
- forgetting that exact-match charts depend on `keyword`-style fields
- trying to chart a field that was ingested as text instead of a number or date

## Why Kibana Becomes So Valuable

Elasticsearch is powerful, but Kibana makes that power usable for day-to-day work. It gives analysts, engineers, and operators a way to inspect data without manually writing every query.

It is also the fastest way to validate whether the rest of the pipeline is producing good data.

## Key Takeaways

- Kibana is the UI layer of the ELK stack.
- It reads data from Elasticsearch.
- Data views define which indexes you work with.
- Discover is for raw exploration.
- Visualizations and dashboards are for analysis and monitoring.

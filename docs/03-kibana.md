# Kibana

> Visualization & Analytics UI

## What is Kibana?

Kibana is the visualization and exploration interface for Elasticsearch data. It lets users search records, build charts, and create dashboards.

Default Kibana web UI port: `5601`

It is commonly used for:

- log exploration
- dashboard creation
- operational monitoring
- security investigation
- ad-hoc data analysis

Kibana is often the first place where users notice whether the underlying data model is clean or messy.

Viewing data in Kibana the fastest way to validate whether the rest of the pipeline is producing good data.

### Kibana's Role

Kibana does not store the main dataset itself. It queries Elasticsearch and presents the results in a user-friendly interface.

That means many Kibana problems are really data problems from earlier in the pipeline.

Examples:

- missing fields
- incorrect datatypes
- timestamps stored as text
- inconsistent field names

## Keywords

Kibana terminology:

- **Data View**: defines which Elasticsearch indexes or index patterns Kibana queries
- **Discover**: explore and search raw data from Elasticsearch
- **Visualization**: represents query results as charts, tables, or other visual formats
- **Dashboard**: a collection of visualizations displayed together for analysis

## Navigating Kibana

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

### Visualizations

Visualizations turn query results into charts such as:

- bar charts
- line charts
- pie charts
- tables
- metric tiles

Visualizations are built on top of fields and aggregations.

### Dashboards

Dashboards combine multiple visualizations in one view.

A dashboard might show:

- log volume over time
- top services by error count
- recent critical events
- event counts by environment

Good advice is to use Kibana Discover before building dashboards.

Dashboards are useful when you want a repeatable operational view rather than one-off exploration.

Visualizations and dashboards are for analysis and monitoring.

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

## Importance of Field Types

Kibana depends heavily on Elasticsearch datatypes.

Examples:

- a `date` field enables timelines and date histograms
- a `keyword` field works well for exact filters and top-values charts
- numeric fields support averages, sums, and ranges
- a `text` field is useful for reading and searching message content

If a field has the wrong datatype, the visualization options may not make sense.

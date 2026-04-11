# ELK Stack

This repository contains beginner-friendly notes for understanding the ELK stack:

- [01-elasticsearch.md](docs/01-elasticsearch.md)
- [02-logstash.md](docs/02-logstash.md)
- [03-kibana.md](docs/03-kibana.md)
- [04-elk-stack.md](docs/04-elk-stack.md)

## What is the ELK Stack?

ELK stands for:

- **Elasticsearch**: stores and searches data
- **Logstash**: ingests and transforms data
- **Kibana**: visualizes and explores data

Together, they form a pipeline for collecting data, storing it, and displaying it.

## Workflow

Think of the stack like this:

1. Logstash receives data from files, APIs, message queues, or other systems
2. Logstash parses and cleans the data so it has a consistent structure
3. Elasticsearch stores that structured data in indexes
4. Kibana lets you search, filter, visualize, and build dashboards from the data in Elasticsearch

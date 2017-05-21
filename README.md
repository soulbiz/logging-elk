# Logging-ELK

Centralizing server logs to a main ELK server

## Overview

Here we'll try to explain how to centralize your logs using an ELK Stack in our main server.
We can accomplish that escenario with a physical server-host or a virtual machine.

## Setup

We'll need to use the following for our setup:

### ELK - Main Server Setup

* [Logstash](https://www.elastic.co/products/logstash)
* [Elasticsearch](https://www.elastic.co/products/elasticsearch)
* [Kibana](https://www.elastic.co/products/kibana)

### Senders - Client Servers Setup

* [Filebeat](https://www.elastic.co/products/beats/filebeat)

## Technologies

**Filebeat** is a log data shipper agent. It monitors and tails specific log directories or files and send them either to a file, Logstash, Elasticsearch, or any other compatible database or data processor in a JSON format, also adding custom data fields like the log-line source or the sender's hostname.

**Logstash** is an open source tool for collecting, parsing, and storing logs for future use. It can unify data from different input sources and normalize it to be sent to Elasticsearch, MongoDB, a file or more destinations. It acts like a pipeline, processing the data with filters and transforming it on the process for further analysis or storage.

**Kibana** is an analytics and visualization platform that can be used to search and view the data that Elasticsearch has indexed. It has a browser-based interface that enables you to check your data or queries in real time and generate custom graphics.

**Elasticsearch** is the main search engine in our setup. Backed by Apache Lucene (an information retrieval software library), it is a full-text search and analytics engine which allows to store, search and analyze big volumes of data efficiently. Its structure makes it more efficient than text-based common databases such as MongoDB. It can work as a single server (node) or a cluster (collection of many nodes).

The Elasticsearch structure works as follows:

* Index: A collection of documents with similar characteristics.
* Type: A partition/category of an index. Used to define a set of documents with mostly common fields.
* Document : Basic information unit. It is represented in JSON, and can contain many data fields.

We can also divide an index storage through "shards" that works as an independent index, or make replicas to prevent data loss!

## Final Infrastructure

![infrastructure](images/elk-infrastructure.png "Main ELK Infrastructure")
*Extracted from "Digital Ocean - Mitchell Anicas" post on how to install ELK.*

## Data Flow

1. Our Filebeat clients check, read and send our logs's data to Logstash in a JSON format.
2. Logstash parses the received data, adding additional fields based on our filters, and sends it to be indexed in Elasticsearch.
3. Elasticsearch stores our JSON data in a common Filebeat-index, separate by the "type" field depending on the log source and its fields.
4. Kibana queries, reads and translates our stored documents into graphical visualizations for an easier, human-focused data analysis.

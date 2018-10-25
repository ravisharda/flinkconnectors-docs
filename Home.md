## Apache Flink Connectors for Pravega

This wiki describes the connectors to read and write [Pravega](http://pravega.io/) streams with [Apache Flink](http://flink.apache.org/) stream processing applications.

Build end-to-end stream processing pipelines that use Pravega as the stream storage and message bus, and Apache Flink for computation over the streams.   See the [Pravega Concepts](http://pravega.io/docs/pravega-concepts/) page for more information.

## Table of Contents

-- [Overview](#overview)
- [Requirements](#requirements)
- Quick Start
    - [Flink Connector Examples for Pravega](https://github.com/pravega/pravega-samples/tree/master/flink-connector-examples)
    - [Creating a Flink stream processing project](https://github.com/pravega/flink-connectors/wiki/Project-Setup#creating-a-flink-stream-processing-project)
    - [Add the connector dependencies](https://github.com/pravega/flink-connectors/wiki/Project-Setup#add-the-connector-dependencies)
    - [Running/deploying the application](
https://github.com/pravega/flink-connectors/wiki/Project-Setup#running--deploying-the-application)
- [Usage](https://github.com/pravega/flink-connectors/wiki/Configuration)
  - [Configurations](https://github.com/pravega/flink-connectors/wiki/Configuration)
      - [PravegaConfig Class](https://github.com/pravega/flink-connectors/wiki/Configuration#pravegaconfig-class)
      - [Creating PravegaConfig](https://github.com/pravega/flink-connectors/wiki/Configuration#creating-pravegaconfig)
      - [Using PravegaConfig](https://github.com/pravega/flink-connectors/wiki/Configuration#using-pravegaconfig)
      - [Configuration Elements](https://github.com/pravega/flink-connectors/wiki/Configuration#configuration-elements)
      - [Understanding the Default Scope](https://github.com/pravega/flink-connectors/wiki/Configuration#understanding-the-default-scope)
 

[Home](home.md)
    - APACHE FLINK CONNECTORS FOR PRAVEGA
          
- [Quickstart](#quickstart)
   - Instructions to download and setup Flink (refer to Pravega link)
   - Instructions to download and setup Flink (refer to Pravega link)
   - Instructions to create a sample Flink application project that uses Flink connectors to read and write into stream
   - Cover the steps (running from IDE) and (running as Flink job on standalone cluster).
   - System requirements (JDK, Flink version, Pravega version etc.,)
   - Talk about the Maven/JCenter repositories from where the artifacts can be downloaded
   - Provide build instructions to build and install artifacts locally
- [Features](#Features)
- [Streaming]
    - FlinkPravegaReader
        - Reader group configurations
        - Deserialization
        - Checkpoint
        - Watermark/Timestamp extraction
        - Parallelism/reader group coordination
        - Builder API usage
        - StreamCuts
   - FlinkPravegaWriter
       - Standard vs Transaction writer
       - Serialization
       - Pravega client configurations
       - Checkpoint/exactly-once support
       - Event routing
       - Parallelism
       - Builder API usage 
- BATCH
   - FlinkPravegaInputFormat
       - Pravega batch client
       - Input splits
       - Segment range
       - Streamcuts
       - Desrializer
       - Builder API usage
       
   - FlinkPravegaOutputFormat
       - Builder API usage
       - Event routing
       - Serialization

- TABLE API/SQL
    - TableSource
       - Stream/batch support
       - Event/processing time support
       - Builder API usage
       - Format support
       - Table schema definitions
    - TableSink 
      - Append-only table sink
      - Stream/batch support
      - Builder API usage
      - Table schema definitions
      - Event router
      - Format support
- SERIALIZATION
   - PravegaSerializationSchema 
   - PravegaDeSerializationSchema 
       - implementation details
       - interoperability features
       - format support
- EVENT ROUTER
   - PravegaEventRouter
      - Usage in the connectors
      - it's reference with respect to Pravega routing key etc.,)
- METRICS
  - List Pravega metrics that are collected in FlinkPravegaReader and FlinkPravegaWriter
  - how that can be queried from a running job?
- Release Management
   - How to relase
   - Publising Artifacts
- Contributing

## System Requirements

**Java 8** or greater is required.

The connectors currently require Apache Flink version **1.4** or newer.
The latest release of these connectors are linked against Flink release *1.4.0*, which should be compatible with all Flink *1.4.x* versions.

## Releases
_At this time, the master branch links to Pravega as a source-based dependency with a specific commit-id._



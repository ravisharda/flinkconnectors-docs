## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Quick Start](#quickstart)
   - Instructions to download and setup Flink (refer to Pravega link)
        - [Flink Connector Examples for Pravega](https://github.com/pravega/pravega-samples/tree/master/flink-connector-examples)
        - [Creating a Flink stream processing project](https://github.com/pravega/flink-connectors/wiki/Project-Setup#creating-a-flink-stream-processing-project)
        - [Add the connector dependencies](https://github.com/pravega/flink-connectors/wiki/Project-Setup#add-the-connector-dependencies)
        - [Running/deploying the application](
https://github.com/pravega/flink-connectors/wiki/Project-Setup#running--deploying-the-application)
   - Instructions to create a sample Flink application project that uses Flink connectors to read and write into stream
   - Cover the steps (running from IDE) and (running as Flink job on standalone cluster).
   
   - Talk about the Maven/JCenter repositories from where the artifacts can be downloaded
   - Provide build instructions to build and install artifacts locally
- [Features](#Features)
- [Configurations](https://github.com/pravega/flink-connectors/wiki/Configuration)
    - [PravegaConfig Class](https://github.com/pravega/flink-connectors/wiki/Configuration#pravegaconfig-class)
    - [Creating PravegaConfig](https://github.com/pravega/flink-connectors/wiki/Configuration#creating-pravegaconfig)
    - [Using PravegaConfig](https://github.com/pravega/flink-connectors/wiki/Configuration#using-pravegaconfig)
    - [Configuration Elements](https://github.com/pravega/flink-connectors/wiki/Configuration#configuration-elements)
    - [Understanding the Default Scope](https://github.com/pravega/flink-connectors/wiki/Configuration#understanding-the-default-scope)

- [Streaming](Streaming-Connector.md)
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
- Batch
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

- Table API/SQL
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
- Serialization
   - PravegaSerializationSchema 
   - PravegaDeSerializationSchema 
       - implementation details
       - interoperability features
       - format support
- Eevet Router
   - PravegaEventRouter
      - Usage in the connectors
      - it's reference with respect to Pravega routing key etc.,)
- Metrics
  - List Pravega metrics that are collected in FlinkPravegaReader and FlinkPravegaWriter
  - how that can be queried from a running job?
- Release Management
   - How to relase
   - Publising Artifacts
- Contributing



The Flink connector library for Pravega provides a table source and table sink for use with the Flink Table API.  The Table API provides a unified API for both the Flink streaming and batch environment.  See the below sections for details.

## Table of Contents
- [Table Source](#table-source)
  - [Parameters](#parameters)
  - [Custom Formats](#custom-formats)
- [Table Sink](#table-sink)
  - [Parameters](#parameters-1)
  - [Custom Formats](#custom-formats-1)

## Table Source
A Pravega stream may be used as a table source within a Flink table program.  The Flink Table API is oriented around Flink's `TableSchema` classes which describe the table fields.  A concrete subclass of `FlinkPravegaTableSource` is then used to parse raw stream data as `Row` objects that conform to the table schema.  The connector library provides out-of-box support for JSON-formatted data with `FlinkPravegaJsonTableSource`, and may be extended to support other formats.

Here's an example of using the provided table source to read JSON-formatted events from a Pravega stream:
```java
// Create a Flink Table environment
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// Load the Pravega configuration
PravegaConfig config = PravegaConfig.fromParams(ParameterTool.fromArgs(args));

// Define the JSON event schema
TableSchema schema = TableSchema.builder()
    .field("sensor_id", Types.STRING())
    .field("temperature", Types.DOUBLE())
    .build();

// Register the Pravega stream as a table
FlinkPravegaJsonTableSource source = FlinkPravegaJsonTableSource.builder()
    .forStream("sensor_stream")
    .withPravegaConfig(config)
    .withSchema(schema)
    .build();
tableEnv.registerTableSource("sensor_reading", source);

// Select event data
Table data = tableEnv.sqlQuery("SELECT * FROM sensor_reading WHERE temperature > 100.0");
```

### Parameters
A builder API is provided to construct an instance of `FlinkPravegaJsonTableSource`.  See the table below for a summary of builder properties.  Note that the builder accepts an instance of `PravegaConfig` for common configuration properties.  See the [Configuration](Configuration) wiki page for more information.

Note that the table source supports both the Flink streaming and batch environments.  In the streaming environment, the table source uses a `FlinkPravegaReader` connector ([ref](Streaming-Connector)); in the batch environment, the table source uses a `FlinkPravegaInputFormat` connector ([ref](Batch-Connector)).  Please see the documentation for the respective connectors to better understand the below parameters.


|Method                |Description|
|----------------------|-----------------------------------------------------------------------|
|`withPravegaConfig`|The Pravega client configuration, which includes connection info, security info, and a default scope.|
|`forStream`|The stream to be read from, with optional start and/or end position.  May be called repeatedly to read numerous streams in parallel.|
|`uid`|The uid to identify the checkpoint state of this source.  _Applies only to streaming API._|
|`withReaderGroupScope`|The scope to store the reader group synchronization stream into.  _Applies only to streaming API._|
|`withReaderGroupName`|The reader group name for display purposes.  _Applies only to streaming API._|
|`withReaderGroupRefreshTime`|The interval for synchronizing the reader group state across parallel source instances.  _Applies only to streaming API._|
|`withCheckpointInitiateTimeout`|The timeout for executing a checkpoint of the reader group state.  _Applies only to streaming API._|
|`withSchema`|The table schema which describes which JSON fields to expect.|
|`failOnMissingField`|A flag indicating whether to fail if a JSON field is missing.|

### Custom Formats
To work with stream events in a format other than JSON, extend `FlinkPravegaTableSource`.  Please look at the implementation of `FlinkPravegaJsonTableSource` ([ref](https://github.com/pravega/flink-connectors/blob/master/src/main/java/io/pravega/connectors/flink/FlinkPravegaJsonTableSource.java)) for details.

## Table Sink
A Pravega stream may be used as an append-only table sink within a Flink table program.  The Flink Table API is oriented around Flink's `TableSchema` classes which describe the table fields.  A concrete subclass of `FlinkPravegaTableSink` is then used to write table rows to a Pravega stream in a particular format.  The connector library provides out-of-box support for JSON-formatted data with `FlinkPravegaJsonTableSource`, and may be extended to support other formats.

Here's an example of using the provided table sink to write JSON-formatted events to a Pravega stream:
```java
// Create a Flink Table environment
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// Load the Pravega configuration
PravegaConfig config = PravegaConfig.fromParams(ParameterTool.fromArgs(args));

// Define a table (see Flink documentation)
Table table = ...

// Write the table to a Pravega stream
FlinkPravegaJsonTableSink sink = FlinkPravegaJsonTableSink.builder()
    .forStream("sensor_stream")
    .withPravegaConfig(config)
    .withRoutingKeyField("sensor_id")
    .withWriterMode(EXACTLY_ONCE)
    .build();
table.writeToSink(sink);
```

### Parameters
A builder API is provided to construct an instance of `FlinkPravegaJsonTableSink`.  See the table below for a summary of builder properties.  Note that the builder accepts an instance of `PravegaConfig` for common configuration properties.  See the [Configuration](Configuration) wiki page for more information.

Note that the table sink supports both the Flink streaming and batch environments.  In the streaming environment, the table sink uses a `FlinkPravegaWriter` connector ([ref](Streaming-Connector)); in the batch environment, the table sink uses a `FlinkPravegaOutputFormat` connector ([ref](Batch-Connector)).  Please see the documentation for the respective connectors to better understand the below parameters.

|Method                |Description|
|----------------------|-----------------------------------------------------------------------|
|`withPravegaConfig`|The Pravega client configuration, which includes connection info, security info, and a default scope.|
|`forStream`|The stream to be written to.|
|`withWriterMode`|The writer mode to provide best-effort, at-least-once, or exactly-once guarantees.|
|`withTxnTimeout`|The timeout for the Pravega transaction that supports the exactly-once writer mode.|
|`withRoutingKeyField`|The table field to use as the routing key for written events.|

### Custom Formats
To work with stream events in a format other than JSON, extend `FlinkPravegaTableSink`.  Please look at the implementation of `FlinkPravegaJsonTableSink` ([ref](https://github.com/pravega/flink-connectors/blob/master/src/main/java/io/pravega/connectors/flink/FlinkPravegaJsonTableSink.java)) for details.

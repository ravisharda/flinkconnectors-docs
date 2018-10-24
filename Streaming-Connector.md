The Flink connector library for Pravega provides a data source and data sink
for use with the Flink Streaming API.   See the below sections for details.

## Table of Contents
- [Data Source (Reader)](#data-source-reader)
  - [Parameters](#parameters)
  - [Input Stream(s)](#input-streams)
  - [Parallelism](#parallelism)
  - [Checkpointing](#checkpointing)
  - [Timestamp Extraction / Watermark Emission](#timestamp-extraction--watermark-emission)
  - [Historical Stream Processing](#historical-stream-processing)
- [Data Sink (Writer)](#data-sink-writer)
  - [Parameters](#parameters-1)
  - [Parallelism](#parallelism-1)
  - [Event Routing](#event-routing)
  - [Event Time Ordering](#event-time-ordering)
  - [Writer Modes](#writer-modes)
- [Data Serialization](#serialization)

## Data Source (Reader)
A Pravega stream may be used as a data source within a Flink streaming program using an instance of `io.pravega.connectors.flink.FlinkPravegaReader`.  The reader reads a given Pravega stream as a [`DataStream`](https://ci.apache.org/projects/flink/flink-docs-release-1.3/api/java/org/apache/flink/streaming/api/datastream/DataStream.html) (the basic abstraction of the Flink Streaming API).

Use the [`StreamExecutionEnvironment::addSource`](https://ci.apache.org/projects/flink/flink-docs-release-1.3/api/java/org/apache/flink/streaming/api/environment/StreamExecutionEnvironment.html#addSource-org.apache.flink.streaming.api.functions.source.SourceFunction-) method
to open a Pravega stream as a DataStream.

Here's an example:
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// Define the Pravega configuration
PravegaConfig config = PravegaConfig.fromParams(params);

// Define the event deserializer
DeserializationSchema<MyClass> deserializer = ...

// Define the data stream
FlinkPravegaReader<MyClass> pravegaSource = FlinkPravegaReader.<MyClass>builder()
    .forStream(...)
    .withPravegaConfig(config)
    .withDeserializationSchema(deserializer)
    .build();
DataStream<MyClass> stream = env.addSource(pravegaSource);
...
```

### Parameters
A builder API is provided to construct an instance of `FlinkPravegaReader`.  See the table below for a summary of builder properties.  Note that the builder accepts an instance of `PravegaConfig` for common configuration properties.  See the [Configuration](Configuration) wiki page for more information.

|Method                |Description|
|----------------------|-----------------------------------------------------------------------|
|`withPravegaConfig`|The Pravega client configuration, which includes connection info, security info, and a default scope.|
|`forStream`|The stream to be read from, with optional start and/or end position.  May be called repeatedly to read numerous streams in parallel.|
|`uid`|The uid to identify the checkpoint state of this source.|
|`withReaderGroupScope`|The scope to store the reader group synchronization stream into.|
|`withReaderGroupName`|The reader group name for display purposes.|
|`withReaderGroupRefreshTime`|The interval for synchronizing the reader group state across parallel source instances.|
|`withCheckpointInitiateTimeout`|The timeout for executing a checkpoint of the reader group state.|
|`withDeserializationSchema`|The deserialization schema which describes how to turn byte messages into events.|

### Input Stream(s)
Each stream in Pravega is contained by a scope.  A scope acts as a namespace for one or more streams.  The `FlinkPravegaReader` is able to read from numerous streams in parallel, even across scopes.  The builder API accepts both _qualified_ and _unqualified_ stream names.  By qualified, we mean that the scope is explicitly specified, e.g. `my-scope/my-stream`.   Unqualified stream names are assumed to refer to the default scope as set in the `PravegaConfig`.

A stream may be specified in one of three ways:
1. As a string containing a qualified name, in the form `scope/stream`.
2. As a string containing an unqualified name, in the form `stream`.  Such streams are resolved to the default scope.
3. As an instance of `io.pravega.client.stream.Stream`, e.g. `Stream.of("my-scope", "my-stream")`.

### Stream Cuts
A `StreamCut` represents a specific position in a Pravega stream, which may be obtained from various API interactions with the Pravega client.   The `FlinkPravegaReader` accepts a `StreamCut` as the start and/or end position of a given stream.

### Parallelism
The data source supports parallelization.  Use the `setParallelism` method to configure the number of parallel instances to execute.  The parallel instances consume the stream in a coordinated manner, each consuming one or more stream segments.

_Note: Coordination is achieved with the use of a Pravega reader group, which is based on a [State Synchronizer](http://pravega.io/docs/pravega-concepts/#state-synchronizers).  The synchronizer creates a backing stream that may be manually deleted after the job finishes._

### Checkpointing
The reader is compatible with Flink checkpoints and savepoints. The reader automatically recovers from failure by rewinding to the checkpointed position in the stream.

A savepoint is self-contained; it contains all information needed to resume from the correct position.

### Timestamp Extraction / Watermark Emission
Pravega is not event time-aware and does not store per-event timestamps or watermarks.
Nonetheless it is possible to use event time semantics via an application-specific timestamp assigner & watermark generator as described in [Flink documentation](https://ci.apache.org/projects/flink/flink-docs-release-1.3/dev/event_timestamps_watermarks.html#timestamp-assigners--watermark-generators).  Specify
an `AssignerWithPeriodicWatermarks` or `AssignerWithPunctuatedWatermarks` on the `DataStream` as normal.

Each parallel instance of the source processes one or more stream segments in parallel. Each watermark generator instance will receive events multiplexed from numerous segments.  Be aware that segments are processed in parallel, and that no effort is made to order the events across segments in terms of their event time.  Also, a given segment may be reassigned to another parallel instance at any time, preserving exactly-once behavior but causing further spread in observed event times.

### Historical Stream Processing
Historical processing refers to processing stream data from a specific position in the stream rather than from the stream's tail.  The builder API provides an overloaded method `forStream` that accepts a `StreamCut` parameter for this purpose. 

## Data Sink (Writer)
A Pravega stream may be used as a data sink within a Flink program using an instance of `io.pravega.connectors.flink.FlinkPravegaWriter`

Use the [`DataStream::addSink`](https://ci.apache.org/projects/flink/flink-docs-release-1.3/api/java/org/apache/flink/streaming/api/datastream/DataStream.html#addSink-org.apache.flink.streaming.api.functions.sink.SinkFunction-) method
to add an instance of the writer to the dataflow program.

Here's an example:
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// Define the Pravega configuration
PravegaConfig config = PravegaConfig.fromParams(params);

// Define the event serializer
SerializationSchema<MyClass> serializer = ...

// Define the event router for selecting the routing key
PravegaEventRouter<MyClass> router = ...

// Define the sink function
FlinkPravegaWriter<MyClass> pravegaSink = FlinkPravegaWriter.<MyClass>builder()
   .forStream(...)
   .withPravegaConfig(config)
   .withSerializationSchema(serializer)
   .withEventRouter(router)
   .withWriterMode(EXACTLY_ONCE)
   .build();

DataStream<MyClass> stream = ...
stream.addSink(pravegaSink);
```

### Parameters
A builder API is provided to construct an instance of `FlinkPravegaReader`.  See the table below for a summary of builder properties.  Note that the builder accepts an instance of `PravegaConfig` for common configuration properties.  See the [Configuration](Configuration) wiki page for more information.

|Method                |Description|
|----------------------|-----------------------------------------------------------------------|
|`withPravegaConfig`|The Pravega client configuration, which includes connection info, security info, and a default scope.|
|`forStream`|The stream to be written to.|
|`withWriterMode`|The writer mode to provide best-effort, at-least-once, or exactly-once guarantees.|
|`withTxnLeaseRenewalPeriod`|The transaction lease renewal period that supports the exactly-once writer mode.|
|`withSerializationSchema`|The serialization schema which describes how to turn events into byte messages.|
|`withEventRouter`|The router function which determines the routing key for a given event.|

### Parallelism
The data sink supports parallelization.  Use the `setParallelism` method to configure the number of parallel instances to execute.

### Event Routing
Every event written to a Pravega stream has an associated routing key.  The routing key is the basis for event ordering.  See the [Pravega documentation](http://pravega.io/docs/pravega-concepts/#events) for details.

To establish the routing key for each event, provide an implementation of `io.pravega.connectors.flink.PravegaEventRouter` when constructing the writer.

### Event Time Ordering
For programs that use Flink's event time semantics, the connector library supports writing events in event time order.   In combination with a routing key, this establishes a well-understood ordering for each key in the output stream.

Use the `FlinkPravegaUtils::writeToPravegaInEventTimeOrder` method to write a given `DataStream` to a Pravega stream such that events are automatically ordered by event time (on a per-key basis).

### Writer Modes
Writer modes relate to guarantees about the persistence of events emitted by the sink to a Pravega stream.  The writer supports three writer modes:
1. **Best-effort** - Any write failures will be ignored hence there could be data loss.
2. **At-least-once** - All events are persisted in Pravega.  Duplicate events
are possible, due to retries or in case of failure and subsequent recovery.
3. **Exactly-once** - All events are persisted in Pravega using a transactional approach integrated with the Flink checkpointing feature.

See the [Pravega documentation](http://pravega.io/docs/pravega-concepts/#transactions) for details on transactional behavior.

## Serialization
_See the [Data Serialization](Data-Serialization) wiki page for more information._


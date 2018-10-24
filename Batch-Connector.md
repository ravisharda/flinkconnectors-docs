The Flink connector library for Pravega makes it possible to use a Pravega stream as a data source in a batch program.  See the below sections for details.

## Table of Contents
- [Data Source (InputFormat)](#data-source-inputformat)
  - [Parameters](#parameters)
  - [Input Stream(s)](#input-streams)
  - [Stream Cuts](#stream-cuts)
  - [Parallelism](#parallelism)
- [Serialization](#serialization)

## Data Source (InputFormat)
A Pravega stream may be used as a data source within a Flink batch program using an instance of `io.pravega.connectors.flink.FlinkPravegaInputFormat`.  The input format reads a given Pravega stream as a [`DataSet`](https://ci.apache.org/projects/flink/flink-docs-release-1.3/api/java/org/apache/flink/api/scala/DataSet.html) (the basic abstraction of the Flink Batch API).   Note that the stream elements are considered to be _unordered_ in the batch programming model.

Use the [`ExecutionEnvironment::createInput`](https://ci.apache.org/projects/flink/flink-docs-release-1.3/api/java/org/apache/flink/api/java/ExecutionEnvironment.html#createInput-org.apache.flink.api.common.io.InputFormat-) method
to open a Pravega stream as a DataSet.

Here's an example:
```java
// Define the Pravega configuration
PravegaConfig config = PravegaConfig.fromParams(params);

// Define the event deserializer 
DeserializationSchema<EventType> deserializer = ...

// Define the input format based on a Pravega stream
FlinkPravegaInputFormat<EventType> inputFormat = FlinkPravegaInputFormat.<EventType>builder()
    .forStream(...)
    .withPravegaConfig(config)
    .withDeserializationSchema(deserializer)
    .build();

DataSource<EventType> dataSet = env.createInput(inputFormat, TypeInformation.of(EventType.class)
                                   .setParallelism(2);
...
```

### Parameters
A builder API is provided to construct an instance of `FlinkPravegaInputFormat`.  See the table below for a summary of builder properties.  Note that the builder accepts an instance of `PravegaConfig` for common configuration properties.  See the [Configuration](Configuration) wiki page for more information.

|Method                |Description|
|----------------------|-----------------------------------------------------------------------|
|`withPravegaConfig`|The Pravega client configuration, which includes connection info, security info, and a default scope.|
|`forStream`|The stream to be read from, with optional start and/or end position.  May be called repeatedly to read numerous streams in parallel.|
|`withDeserializationSchema`|The deserialization schema which describes how to turn byte messages into events.|

### Input Stream(s)
Each stream in Pravega is contained by a scope.  A scope acts as a namespace for one or more streams.  The `FlinkPravegaReader` is able to read from numerous streams in parallel, even across scopes.  The builder API accepts both _qualified_ and _unqualified_ stream names.  By qualified, we mean that the scope is explicitly specified, e.g. `my-scope/my-stream`.   Unqualified stream names are assumed to refer to the default scope as set in the `PravegaConfig`.

A stream may be specified in one of three ways:
1. As a string containing a qualified name, in the form `scope/stream`.
2. As a string containing an unqualified name, in the form `stream`.  Such streams are resolved to the default scope.
3. As an instance of `io.pravega.client.stream.Stream`, e.g. `Stream.of("my-scope", "my-stream")`.

### Stream Cuts
A `StreamCut` represents a specific position in a Pravega stream, which may be obtained from various API interactions with the Pravega client.   The `FlinkPravegaReader` accepts a `StreamCut` as the start and/or end position of a given stream.

If unspecified, the default start position is the earliest available data.  The default end position is all available data as of when job execution begins.

### Parallelism
The data source supports parallelization.  Use the `setParallelism` method to configure the number of parallel instances to execute.  The parallel instances consume the stream in a coordinated manner, each consuming one or more stream segments.

## Serialization
_See the [Data Serialization](Data-Serialization) wiki page for more information._

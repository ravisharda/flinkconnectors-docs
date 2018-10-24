Building the connectors from source is only necessary when you want to use or contribute to the latest (not yet released) version of the Pravega Flink connectors.

The connector project is linked to a specific version of Pravega, based on a git submodule pointing to a commit-id.  Gradle automatically builds Pravega as necessary to build the connector.

## Build Steps
### Clone the flink-connectors Repository
Be sure to use the `--recursive` option when cloning the repository.   Alternately, update the submodules using:
```
$ git submodule update
```

### Build and install the Flink Connector library

Build and install the latest version of the Pravega Flink Connector library into your local Maven repository.

```
$ git clone https://github.com/pravega/flink-connectors.git
$ ./gradlew clean install
```

The resulting jar file will be called `pravega-connectors-flink_2.11-<version>.jar`.

## Customizing the Build
### Building against a custom Flink version

You can check and change the Flink version that Pravega builds against via the `flinkVersion` variable in the `gradle.properties` file.
Note that you can only choose Flink versions that are compatible with the latest connector code.

### Building against another Scala version

This section is only relevant if you use [Scala](https://www.scala-lang.org/) in the stream processing application in with Flink and Pravega.

Parts of the Apache Flink use the language or depend on libraries written in Scala. Because Scala is not strictly compatible across versions, there exist different versions of Flink compiled for different Scala versions.
If you use Scala code in the same application where you use the Apache Flink or the Flink connectors, you typically have to make sure you use a version of Flink that uses the same Scala version as your application.

By default, the dependencies point to Flink for Scala **2.11**.
To depend on released Flink artifacts for a different Scala version, you need to edit the `build.gradle` file and change all entries for the Flink dependencies to have a different Scala version suffix. For example, `flink-streaming-java_2.11` would be replaced by `flink-streaming-java_2.12` for Scala 2.12.

In order to build a new version of Flink for a different Scala version, please refer to the [Flink documentation](https://ci.apache.org/projects/flink/flink-docs-release-1.4/start/building.html#scala-versions) (the link refers to Flink 1.4).

## IDE Setup (IntelliJ IDEA)
Import the project into IDEA as a Gradle-based project.

Please install the Lombok plugin to be able to build from within the IDE.  The source code uses Lombok annotations to generate various helper code.

## Updating the Pravega dependency

1. Update the submodule to the head of the master branch:
```
git submodule update --remote --merge
```

2. Build and test the connector.

3. Commit the changes, including the updated commit-id.

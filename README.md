# Apache Paimon Trino Connector

[![License](https://img.shields.io/badge/license-Apache%202-4EB1BA.svg)](https://www.apache.org/licenses/LICENSE-2.0.html)
[![Trino Version](https://img.shields.io/badge/Trino-440-blue.svg)](https://trino.io/)
[![Paimon Version](https://img.shields.io/badge/Paimon-1.2.0-green.svg)](https://paimon.apache.org/)
[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.java.net/)
[![Maven Central](https://img.shields.io/maven-central/v/org.apache.paimon/paimon-trino-440.svg)](https://search.maven.org/artifact/org.apache.paimon/paimon-trino-440)
[![Build Status](https://img.shields.io/github/actions/workflow/status/tchivs/paimon-trino/build.yml?branch=main)](https://github.com/tchivs/paimon-trino/actions)
[![Code Style](https://img.shields.io/badge/code%20style-Google%20Java%20Format-brightgreen.svg)](https://github.com/google/google-java-format)
[![Checkstyle](https://img.shields.io/badge/checkstyle-8.45.1-blue.svg)](https://checkstyle.sourceforge.io/)

This repository is a Trino Connector for the [Apache Paimon](https://paimon.apache.org/) project.

## About

Apache Paimon is an open source project of [The Apache Software Foundation](https://apache.org/) (ASF). This connector enables seamless integration between Trino and Apache Paimon, allowing you to query Paimon tables using Trino's powerful distributed SQL engine.

## Features

✅ **Full Type Support** - Complete implementation of complex types (Array, Map, Row, Variant)
✅ **Time Travel** - Query historical snapshots and tags
✅ **CDC Support** - Access change data capture streams
✅ **Predicate Pushdown** - Efficient filtering with partition and file-level pruning
✅ **Schema Evolution** - Handle schema changes seamlessly
✅ **Write Operations** - Support for INSERT and INSERT OVERWRITE
✅ **Merge Operations** - UPDATE and DELETE via Trino's MERGE syntax
✅ **ORC Optimization** - Direct ORC reading for improved performance

## Requirements

- Java 21 or higher
- Trino 440
- Apache Paimon 1.2.0
- Maven 3.6.0+ (for building from source)

## Quick Start

### Installation

1. Download the latest release from [GitHub Releases](https://github.com/tchivs/paimon-trino/releases)

2. Extract the plugin to your Trino plugin directory:

```bash
cd /path/to/trino/plugin
tar -xzf paimon-trino-440-1.2.0-plugin.tar.gz
```

3. Configure the connector by creating `etc/catalog/paimon.properties`:

```properties
connector.name=paimon
warehouse=file:///tmp/paimon
```

4. Restart Trino server

### Basic Usage

```sql
-- Create a catalog
CREATE SCHEMA paimon.test_db;

-- Create a table
CREATE TABLE paimon.test_db.users (
  id INT,
  name VARCHAR,
  age INT
) WITH (
  primary-key = ARRAY['id']
);

-- Insert data
INSERT INTO paimon.test_db.users VALUES
  (1, 'Alice', 30),
  (2, 'Bob', 25);

-- Query data
SELECT * FROM paimon.test_db.users;

-- Time travel query
SELECT * FROM paimon.test_db.users FOR VERSION AS OF 1;
```

## Building from Source

### Prerequisites

**IMPORTANT**: This project depends on the parent POM from the main Apache Paimon project. Before building paimon-trino, you must:

1. Clone and build the main Paimon repository:
```bash
git clone https://github.com/apache/paimon.git
cd paimon
mvn clean install -DskipTests
```

2. Build paimon-trino:
```bash
git clone https://github.com/tchivs/paimon-trino.git
cd paimon-trino
mvn clean package
```

The plugin package will be created at:
```
target/paimon-trino-440-1.2-SNAPSHOT-plugin.tar.gz
```

### Build Commands

```bash
# Build the project
mvn clean package

# Build without tests
mvn clean package -DskipTests

# Run specific test
mvn test -Dtest=TrinoITCase#testProjection

# Apply code formatting
mvn spotless:apply

# Check code style
mvn checkstyle:check
```

## Configuration

### Catalog Properties

| Property | Description | Default |
|----------|-------------|---------|
| `connector.name` | Must be `paimon` | - |
| `warehouse` | Warehouse location (file:// or s3://) | - |
| `hive.config.resources` | Hadoop configuration files | - |

### Session Properties

```sql
-- Set snapshot for time travel
SET SESSION paimon.snapshot = 1;

-- Set tag for time travel
SET SESSION paimon.tag = 'tag-name';
```

## Deployment

### Standard Deployment

1. Extract the plugin archive to Trino's plugin directory:
```bash
cd /data/trino/plugin/
tar -xzf paimon-trino-440-1.2.0-plugin.tar.gz
```

2. Verify the deployment:
```bash
ls -la /data/trino/plugin/paimon/
# Should show 318 JAR files including:
# - paimon-trino-440-1.2.0.jar
# - guice-7.0.0.jar
# - bootstrap-241.jar
# - ... and all other dependencies
```

3. Configure catalog properties in `etc/catalog/paimon.properties`

4. Restart Trino

### Common Deployment Issues

**ClassNotFoundException: com/google/inject/Key**
- **Cause**: Only the main JAR was copied instead of the full plugin archive
- **Solution**: Extract the complete tar.gz file to include all 318 dependency JARs

## Architecture

### Core Components

- **TrinoPlugin** - Entry point registered in `META-INF/services/io.trino.spi.Plugin`
- **TrinoConnectorFactory** - Creates connector instances with Guice dependency injection
- **TrinoMetadata** - Implements DDL operations and pushdown optimizations
- **TrinoSplitManager** - Generates splits for parallel processing
- **TrinoPageSourceProvider** - Creates page sources (supports direct ORC reading)
- **TrinoPageSinkProvider** - Handles INSERT operations

### Data Flow

**Read Path**: SplitManager → PageSource (ORC direct/Paimon readers) → Deletion vectors/File indexes
**Write Path**: PageSink → BatchTableWrite → Commit messages → TrinoMetadata.finishInsert()
**Merge Path**: MergePageSourceWrapper → MergeSink → DELETE_ROW_AND_INSERT_ROW

## Development

### Code Quality

This project uses:
- **Google Java Format** (AOSP style) - `mvn spotless:apply`
- **Checkstyle 8.45.1** - Strict code style enforcement
- **Import Order**: `org.apache.paimon` → `*` → `javax` → `java` → `scala` → `#`(static)

### Testing

```bash
# Run all tests
mvn test

# Run integration tests
mvn test -Dtest=TrinoITCase

# Run with specific JDK
export JAVA_HOME=/path/to/jdk-21
mvn clean test
```

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes with proper tests
4. Run `mvn spotless:apply` and `mvn checkstyle:check`
5. Submit a pull request

## Version History

### v1.2.0 (2025-11-14)

- ✨ Complete implementation of complex type handling (Array, Map, Row, Variant)
- ⬆️ Upgraded Checkstyle from 8.14 to 8.45.1
- ✨ Updated to Paimon 1.2.0 API compatibility
- 🐛 Fixed Jackson serialization for Trino connector classes
- ♻️ Improved code organization with 21% reduction in duplication
- 📝 Enhanced Javadoc compliance

See [CHANGES_SUMMARY.md](CHANGES_SUMMARY.md) for detailed changelog.

## Documentation

- [Quick Start Guide](QUICKSTART.md) - Get started quickly
- [Release Guide](RELEASE_GUIDE.md) - Release process documentation
- [CLAUDE.md](CLAUDE.md) - Development guidelines for Claude Code

## Resources

- [Apache Paimon](https://paimon.apache.org/) - Official Paimon website
- [Trino Documentation](https://trino.io/docs/current/) - Trino SQL engine docs
- [GitHub Issues](https://github.com/tchivs/paimon-trino/issues) - Report bugs or request features

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

```
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
```

## Acknowledgments

- Apache Paimon Team - For the excellent data lake platform
- Trino Community - For the powerful distributed SQL engine
- Contributors - Thank you to all who have contributed to this project

---

Made with ❤️ by the Paimon-Trino community

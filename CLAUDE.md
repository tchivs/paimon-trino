# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About

Apache Paimon Trino Connector is a Trino plugin that enables querying Apache Paimon tables. This is a Maven-based Java project targeting Java 21 and Trino version 440.

## Build Prerequisites

**IMPORTANT**: This project depends on the parent POM from the main Apache Paimon project. Before building paimon-trino, you must:

1. Clone and build the main Paimon repository:
```bash
git clone https://github.com/apache/paimon.git
cd paimon
mvn clean install -DskipTests
```

2. Then build paimon-trino:
```bash
cd ../paimon-trino
mvn clean install -Papache-build
```

Alternatively, use the `apache-build` profile to fetch dependencies from Apache's snapshot repository (may not always have latest snapshots):
```bash
mvn clean install -Papache-build -DskipTests
```

## Build and Test Commands

### Building
```bash
# Build the project (runs checkstyle, spotless, tests)
mvn clean package

# Build without tests
mvn clean package -DskipTests

# Build plugin assembly (creates distributable plugin)
mvn clean package assembly:attached

# Code formatting with Spotless
mvn spotless:apply

# Check code style
mvn checkstyle:check
```

### Testing
```bash
# Run all tests
mvn test

# Run a specific test class
mvn test -Dtest=TrinoITCase

# Run a specific test method
mvn test -Dtest=TrinoITCase#testMethodName
```

### Code Quality
```bash
# Apply code formatting (Google Java Format with AOSP style)
mvn spotless:apply

# Check formatting without applying
mvn spotless:check

# Run checkstyle validation
mvn checkstyle:check
```

## Code Architecture

### Plugin Entry Point
- `TrinoPlugin`: Entry point registered in `META-INF/services/io.trino.spi.Plugin`
- `TrinoConnectorFactory`: Creates connector instances with dependency injection (Guice)
- Supports reading Hadoop configuration from XML files via `hive.config.resources` property

### Core Components

**Metadata Layer**
- `TrinoMetadata`: Implements Trino's `ConnectorMetadata` interface
  - Handles schema/table DDL operations (create, drop, rename, alter)
  - Manages column operations (add, drop, rename)
  - Supports pushdown optimizations (filter, projection, limit)
  - Implements merge operations for UPDATE/DELETE using `DELETE_ROW_AND_INSERT_ROW` paradigm
- `TrinoCatalog`: Wrapper around Paimon's catalog interface (in `catalog/` package)
  - Provides session initialization
  - Handles database and table operations

**Data Access Layer**
- `TrinoSplitManager`: Generates splits from Paimon table for parallel processing
- `TrinoPageSourceProvider`: Creates page sources for reading data
  - Supports direct ORC reading for performance (bypasses Paimon readers when possible)
  - Implements schema evolution mapping between table and file schemas
  - Handles deletion vectors and file index predicates
  - Falls back to Paimon's native readers for non-ORC formats
- `TrinoPageSinkProvider`: Creates page sinks for writing data
  - Supports INSERT and INSERT OVERWRITE operations
  - Validates bucket modes (HASH_FIXED and BUCKET_UNAWARE supported)

**Type System**
- `TrinoTypeUtils`: Bidirectional conversion between Paimon types and Trino types
- `TrinoColumnHandle`: Represents column metadata with type information

**Filter Processing**
- `TrinoFilterConverter`: Converts Trino's `TupleDomain` to Paimon predicates
- `TrinoFilterExtractor`: Extracts supported filters from Trino constraints
- Filters are pushed down to Paimon for efficient data skipping

**File I/O Integration**
- `fileio/` package: Adapts Paimon's FileIO to Trino's filesystem abstraction
- `TrinoFileIO`, `TrinoInputStreamWrapper`, `TrinoFileStatus`: Bridge between systems

**Advanced Features**
- `functions/tablechanges/`: Table functions for CDC (change data capture)
- `TrinoNodePartitioningProvider`: Supports fixed bucket partitioning for distributed writes
- Time travel queries via Trino's `ConnectorTableVersion` (supports snapshots and tags)

### Data Flow

**Read Path**:
1. `TrinoSplitManager` creates splits from Paimon table
2. `TrinoPageSourceProvider` receives split and creates appropriate page source
3. For ORC files: Uses `DirectTrinoPageSource` with native Trino ORC readers
4. For other formats: Uses `TrinoPageSource` with Paimon readers
5. Deletion vectors and file indexes applied when present

**Write Path**:
1. `TrinoPageSinkProvider` creates `TrinoPageSink` with Paimon's `BatchTableWrite`
2. Data flows through `TrinoPageSink.appendPage()`
3. Commit messages serialized and returned to coordinator
4. `TrinoMetadata.finishInsert()` commits all messages via Paimon's batch writer

**Merge (UPDATE/DELETE) Path**:
1. `TrinoMergePageSourceWrapper` wraps page source to extract row IDs
2. `TrinoMergeSink` separates delete and insert operations
3. Uses same commit flow as write path

## Coding Standards

### Code Formatting
- Uses Spotless with Google Java Format (AOSP style)
- Import order: `org.apache.paimon`, `org.apache.paimon.shaded`, `*`, `javax`, `java`, `scala`, `#` (static)
- Maximum file length: 3000 lines
- Checkstyle enforces strict style rules (see `tools/maven/checkstyle.xml`)

### Import Rules
- Do NOT import `com.google.common.*` directly - use `paimon-shaded-guava` instead
- Avoid `org.apache.commons.lang` - use `commons-lang3`
- No star imports allowed
- Static imports come last, alphabetically sorted

### Naming Conventions
- Package names: lowercase with dots (e.g., `org.apache.paimon.trino`)
- Classes: PascalCase
- Constants: ALL_CAPS_WITH_UNDERSCORES
- Methods/variables: camelCase
- Method names may use underscores for test methods

### Comments and Documentation
- TODO comments must not include usernames: `TODO(username)` is forbidden
- Protected/public methods and classes require Javadoc
- Missing param/return tags allowed in Javadoc

### Special Patterns in This Codebase

**ClassLoader Handling**
- Use `ClassLoaderUtils.runWithContextClassLoader()` when bridging Trino and Paimon APIs
- Critical for serialization/deserialization and plugin isolation

**Session Management**
- Always call `trinoCatalog.initSession(session)` before catalog operations
- Session contains dynamic options and configuration

**Dynamic Options**
- Tables can have dynamic options overlaid at query time (e.g., time travel)
- Use `tableWithDynamicOptions(catalog, session)` to apply session-level overrides

**Bucket Mode Validation**
- Only `HASH_FIXED` and `BUCKET_UNAWARE` modes supported for writes
- Dynamic bucket tables are not yet supported (see TODO comments)

## Testing

### Test Structure
- Unit tests: `*Test.java` in `src/test/java/org/apache/paimon/trino/`
- Integration test: `TrinoITCase.java` (comprehensive query testing)
- `TrinoQueryRunner`: Test harness for running Trino queries
- `TrinoTestUtils`: Common test utilities

### Key Test Files
- `TrinoITCase.java`: Main integration test covering DDL/DML operations
- `TrinoDistributedQueryTest.java`: Tests distributed query execution
- `TrinoFilterConverterTest.java`: Tests predicate pushdown
- `TrinoTypeTest.java`: Tests type conversion

## Common Development Tasks

When implementing new features:
1. Check if Paimon core library supports the feature first
2. Consider whether filters/projections can be pushed down
3. Ensure proper classloader context when calling Paimon APIs
4. Add type conversions in `TrinoTypeUtils` for new types
5. Update `TrinoMetadata` for DDL changes
6. Add integration tests in `TrinoITCase` for end-to-end validation

When debugging:
- Check catalog session initialization
- Verify dynamic options are properly merged
- Examine filter conversion and predicate pushdown
- Review split generation in `TrinoSplitManager`

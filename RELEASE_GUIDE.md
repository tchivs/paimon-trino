# Paimon Trino Connector Release Guide

## Quick Start

### Using GitHub Actions (Recommended)

1. Navigate to **Actions** tab in the GitHub repository
2. Select **"Release Paimon Trino Connector"**
3. Click **"Run workflow"**
4. Enter the parameters:
   ```
   Trino version: 478
   Airlift version: (leave empty for auto-detection)
   Release tag: v1.0.0-trino-478
   Release name: Paimon Trino 478 Connector v1.0.0
   ```
5. Click **"Run workflow"** to start

The workflow will:
- Automatically detect the correct Airlift version for your Trino version
- Update `pom.xml` with the specified versions
- Build the project with Maven
- Create a GitHub release with installation instructions
- Upload plugin ZIP and JAR artifacts

### Manual Build

If you prefer to build manually:

```bash
# 1. Update versions in pom.xml
# Edit trino.version and airlift.version properties

# 2. Build the project
mvn clean package assembly:attached -Papache-build -DskipTests

# 3. Find artifacts in target/ directory
ls -lh target/*.zip target/*.jar
```

## Supported Trino Versions

The workflow has been tested with:

| Trino Version | Airlift Version | Status |
|---------------|-----------------|--------|
| 478 | 369 | ✅ Tested |
| 477 | 369 | ✅ Compatible |
| 476 | 369 | ✅ Compatible |
| 468 | 343 | ✅ Compatible |
| 440 | 241 | ✅ Original version |

## Version Compatibility

### Trino 478 Changes

When upgrading from Trino 440 to 478, the following changes were made:

1. **Trino version**: 440 → 478
2. **Airlift version**: 241 → 369
3. **Parent POM**: Uses airbase 322 (managed by Trino)

The workflow automatically handles these version dependencies.

## Installation

After the release is created:

1. Download the plugin ZIP from the GitHub release page
2. Extract to your Trino plugin directory:
   ```bash
   cd /path/to/trino/plugin
   unzip paimon-trino-478-*.zip
   ```
3. Configure the catalog (`/etc/trino/catalog/paimon.properties`):
   ```properties
   connector.name=paimon
   warehouse=/path/to/warehouse
   ```
4. Restart Trino server

## Troubleshooting

### Common Issues

#### Build fails with missing parent POM

**Solution**: Ensure you're using the `apache-build` profile which fetches from Apache snapshots:
```bash
mvn clean package -Papache-build -DskipTests
```

#### Airlift version mismatch

**Solution**: Check the [Trino release notes](https://trino.io/docs/current/release.html) or let the workflow auto-detect:
```bash
# Download Trino parent POM
curl -s https://repo1.maven.org/maven2/io/trino/trino-root/478/trino-root-478.pom | grep airlift.version
```

#### Tests failing

**Solution**: Use `-DskipTests` to skip tests during release build, or investigate test failures:
```bash
mvn test -Dtest=TrinoITCase
```

### GitHub Actions Issues

#### Workflow not appearing

- Ensure the workflow file is in `.github/workflows/` directory
- Check that your repository has Actions enabled in Settings

#### Permission denied when creating release

- Verify the repository has `contents: write` permission
- Check that `GITHUB_TOKEN` is available (it's automatic)

#### Build timeout

- The workflow has sufficient timeout for most builds
- If it times out, check Maven dependencies are downloading correctly

## Development Workflow

### Testing Changes Locally

Before creating a release:

```bash
# 1. Update versions
sed -i 's/<trino.version>.*<\/trino.version>/<trino.version>478<\/trino.version>/' pom.xml
sed -i 's/<airlift.version>.*<\/airlift.version>/<airlift.version>369<\/airlift.version>/' pom.xml

# 2. Build
mvn clean package -Papache-build -DskipTests

# 3. Verify artifacts
ls -lh target/*.zip
```

### Creating a Pre-release

For testing before official release:

1. Use workflow with tag like `v1.0.0-trino-478-rc1`
2. Mark as pre-release in GitHub
3. Test thoroughly
4. Create final release with `v1.0.0-trino-478`

## Automation Features

The GitHub Actions workflow includes:

✅ **Auto-detection**: Automatically detects correct Airlift version from Trino POM
✅ **Version update**: Updates `pom.xml` with specified versions
✅ **Build**: Compiles and packages with Maven
✅ **Artifacts**: Creates plugin ZIP and JAR files
✅ **Release notes**: Generates detailed installation instructions
✅ **Upload**: Uploads artifacts to GitHub release
✅ **Caching**: Uses Maven cache for faster builds

## Advanced Configuration

### Customizing the Build

Edit `.github/workflows/release.yml` to:

- Add additional build profiles
- Include more artifacts
- Customize release notes format
- Add integration tests before release
- Deploy to Maven Central (requires credentials)

### Version Mapping

The workflow includes a fallback version mapping table for auto-detection. Update it as new Trino versions are released:

```yaml
case $TRINO_VERSION in
  478) AIRLIFT_VERSION="369" ;;
  477) AIRLIFT_VERSION="369" ;;
  # Add new versions here
  *) AIRLIFT_VERSION="369" ;;
esac
```

## Contributing

When adding support for new Trino versions:

1. Test the build locally first
2. Update the version mapping in the workflow
3. Update this documentation with compatibility info
4. Create a test release to verify automation

## Resources

- [Trino Documentation](https://trino.io/docs/current/)
- [Trino Release Notes](https://trino.io/docs/current/release.html)
- [Apache Paimon](https://paimon.apache.org/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)

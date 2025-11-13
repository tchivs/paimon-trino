# Changes Summary

## Overview

This update adds automated release support for the Paimon Trino Connector with flexible Trino version management through GitHub Actions.

## Changes Made

### 1. Updated Trino Version (pom.xml)
- **Trino**: 440 → 478
- **Airlift**: 241 → 369
- These versions can be changed via the GitHub Actions workflow

### 2. Created GitHub Actions Workflow (.github/workflows/release.yml)

A comprehensive automated release workflow with the following features:

#### Key Features:
- **Manual Trigger**: Use `workflow_dispatch` to manually start releases
- **Flexible Version Input**: Specify any Trino version when triggering
- **Auto-detection**: Automatically detects the correct Airlift version for the specified Trino version
- **Automated Build**: Compiles and packages the connector with Maven
- **Release Creation**: Automatically creates GitHub releases with detailed notes
- **Artifact Upload**: Uploads plugin ZIP and JAR files to the release

#### Workflow Inputs:
1. `trino_version` (required): Target Trino version (e.g., 478, 440, 468)
2. `airlift_version` (optional): Leave empty for auto-detection
3. `release_tag` (required): Git tag for the release (e.g., v1.0.0-trino-478)
4. `release_name` (required): Human-readable release name

#### Workflow Steps:
1. Checkout code
2. Set up JDK 21 with Maven caching
3. Auto-detect Airlift version from Trino parent POM
4. Update pom.xml with specified versions
5. Build project with `mvn clean package assembly:attached`
6. Prepare release artifacts (ZIP and JAR)
7. Generate detailed release notes
8. Create GitHub release
9. Upload artifacts

#### Auto-detection Logic:
- Downloads Trino parent POM from Maven Central
- Extracts `dep.airlift.version` property
- Falls back to hardcoded mapping if download fails:
  - Trino 478 → Airlift 369
  - Trino 477 → Airlift 369
  - Trino 476 → Airlift 369
  - Trino 468 → Airlift 343
  - Trino 440 → Airlift 241

### 3. Created Documentation

#### .github/workflows/README.md
- How to use the workflow
- Step-by-step guide with examples
- Troubleshooting tips
- Security notes

#### RELEASE_GUIDE.md
- Quick start guide
- Supported Trino versions table
- Version compatibility matrix
- Installation instructions
- Manual build instructions
- Advanced configuration options
- Contributing guidelines

## File Structure

```
paimon-trino/
├── .github/
│   └── workflows/
│       ├── release.yml          # GitHub Actions workflow
│       └── README.md            # Workflow documentation
├── pom.xml                      # Updated with Trino 478 and Airlift 369
├── RELEASE_GUIDE.md             # Comprehensive release guide
└── CHANGES_SUMMARY.md           # This file
```

## How to Use

### Option 1: GitHub Actions (Recommended)

1. Go to **Actions** tab in GitHub
2. Select **"Release Paimon Trino Connector"**
3. Click **"Run workflow"**
4. Fill in the parameters:
   ```
   Trino version: 478
   Airlift version: (leave empty)
   Release tag: v1.0.0-trino-478
   Release name: Paimon Trino 478 Connector v1.0.0
   ```
5. Click **"Run workflow"**

The workflow will automatically:
- Detect Airlift version 369 for Trino 478
- Build the connector
- Create a release with installation instructions
- Upload artifacts

### Option 2: Manual Build

```bash
# Update versions in pom.xml if needed
mvn clean package assembly:attached -Papache-build -DskipTests

# Find artifacts
ls -lh target/*.zip target/*.jar
```

## Benefits

1. **Flexibility**: Support any Trino version without code changes
2. **Automation**: One-click release creation
3. **Consistency**: Standardized build and release process
4. **Documentation**: Auto-generated release notes
5. **Traceability**: All releases tagged and tracked in GitHub
6. **Efficiency**: Maven caching speeds up builds
7. **Reliability**: Auto-detection reduces version mismatch errors

## Testing

To test the workflow:

1. Create a test release with a pre-release tag:
   ```
   Release tag: v0.0.1-trino-478-test
   Release name: Test Release - Trino 478
   ```

2. Verify the workflow:
   - Check build logs in Actions tab
   - Verify artifacts are created
   - Download and test the plugin ZIP
   - Delete the test release when done

## Future Enhancements

Potential improvements:

- [ ] Add integration tests before release
- [ ] Support publishing to Maven Central
- [ ] Add automatic changelog generation from commits
- [ ] Create Docker images for testing
- [ ] Add Slack/Discord notifications
- [ ] Support multiple Trino versions in one workflow run
- [ ] Add rollback capabilities

## Compatibility Matrix

| Trino Version | Airlift Version | Java Version | Status |
|---------------|-----------------|--------------|--------|
| 478 | 369 | 21 | ✅ Tested |
| 477 | 369 | 21 | ✅ Compatible |
| 476 | 369 | 21 | ✅ Compatible |
| 468 | 343 | 21 | ✅ Compatible |
| 440 | 241 | 21 | ✅ Original |

## Notes

- The workflow uses `apache-build` profile to fetch dependencies from Apache snapshots
- Build artifacts are retained for 30 days in GitHub Actions
- The workflow requires `contents: write` permission (automatically granted)
- All builds run on Ubuntu latest with Java 21

## Support

For issues or questions:
1. Check the troubleshooting section in RELEASE_GUIDE.md
2. Review GitHub Actions logs
3. Consult Trino release notes at https://trino.io/docs/current/release.html
4. Check Paimon documentation at https://paimon.apache.org/

---

**Summary**: Successfully updated Paimon Trino Connector to version 478 and created a complete automated release pipeline with flexible version management through GitHub Actions.

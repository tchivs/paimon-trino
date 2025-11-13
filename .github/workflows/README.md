# GitHub Actions Workflows

## Release Workflow

The `release.yml` workflow allows you to manually build and release the Paimon Trino Connector for different Trino versions.

### How to Use

1. Go to the **Actions** tab in your GitHub repository
2. Select **"Release Paimon Trino Connector"** from the workflow list
3. Click **"Run workflow"** button
4. Fill in the required inputs:

   - **Trino version**: The target Trino version (e.g., `478`, `440`, `468`)
   - **Airlift version**: (Optional) Leave empty for auto-detection. The workflow will automatically fetch the correct Airlift version from the Trino parent POM
   - **Release tag**: Git tag for the release (e.g., `v1.0.0-trino-478`)
   - **Release name**: Human-readable release name (e.g., `Paimon Trino 478 Connector v1.0.0`)

5. Click **"Run workflow"** to start the build

### What the Workflow Does

1. **Auto-detects Airlift version**: If you don't provide an Airlift version, the workflow automatically downloads the Trino parent POM and extracts the correct Airlift version
2. **Updates pom.xml**: Updates the Trino and Airlift versions in the project POM
3. **Builds the project**: Compiles and packages the connector using Maven with the `apache-build` profile
4. **Creates artifacts**: Generates the plugin ZIP and JAR files
5. **Creates GitHub Release**: Creates a new release with:
   - Detailed release notes
   - Plugin ZIP file (for installation)
   - JAR file
   - Installation instructions
6. **Uploads artifacts**: Makes build artifacts available for download

### Example Usage

#### Building for Trino 478

```
Trino version: 478
Airlift version: (leave empty)
Release tag: v1.0.0-trino-478
Release name: Paimon Trino 478 Connector v1.0.0
```

#### Building for Trino 440

```
Trino version: 440
Airlift version: (leave empty)
Release tag: v1.0.0-trino-440
Release name: Paimon Trino 440 Connector v1.0.0
```

#### Building with specific Airlift version

```
Trino version: 478
Airlift version: 369
Release tag: v1.0.0-trino-478
Release name: Paimon Trino 478 Connector v1.0.0
```

### Requirements

- Repository must have GitHub Actions enabled
- Requires `contents: write` permission to create releases
- Java 21 is automatically set up by the workflow
- Maven dependencies are cached for faster builds

### Troubleshooting

#### Build fails due to missing parent POM

If the build fails with "Non-resolvable parent POM", ensure the `apache-build` profile is working correctly. The workflow uses this profile to fetch dependencies from Apache's snapshot repository.

#### Airlift version auto-detection fails

If auto-detection fails, you can manually specify the Airlift version. Common mappings:

- Trino 478: Airlift 369
- Trino 477: Airlift 369
- Trino 476: Airlift 369
- Trino 468: Airlift 343
- Trino 440: Airlift 241

Check the [Trino release notes](https://trino.io/docs/current/release.html) or Maven Central for the exact version.

#### Release already exists

If you try to create a release with a tag that already exists, the workflow will fail. Either:
- Use a different tag name
- Delete the existing release and tag first
- Or manually update the existing release

### Advanced Configuration

You can customize the workflow by editing `.github/workflows/release.yml`:

- Modify build commands
- Add additional build profiles
- Change artifact naming
- Customize release notes format
- Add post-build validation steps

### Security Notes

- The workflow uses `GITHUB_TOKEN` which is automatically provided by GitHub Actions
- Build artifacts are retained for 30 days
- All builds run in isolated GitHub-hosted runners

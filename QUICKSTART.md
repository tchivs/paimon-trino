# Quick Start - Paimon Trino Connector Release

## Creating a Release (GitHub Actions)

### Step 1: Navigate to Actions
Go to your GitHub repository → **Actions** tab

### Step 2: Select Workflow
Click on **"Release Paimon Trino Connector"** in the left sidebar

### Step 3: Run Workflow
Click the **"Run workflow"** button (top right)

### Step 4: Fill Parameters
```
Trino version:    478
Airlift version:  [leave empty]
Release tag:      v1.0.0-trino-478
Release name:     Paimon Trino 478 Connector v1.0.0
```

### Step 5: Start Build
Click **"Run workflow"** button at the bottom

### Step 6: Wait for Completion
- Build takes ~5-10 minutes
- Check progress in Actions tab
- Release appears in Releases section when done

## What You Get

After the workflow completes:

✅ **GitHub Release** with:
- Plugin ZIP file for installation
- JAR file
- Detailed release notes
- Installation instructions

✅ **Artifacts** available for 30 days in Actions tab

✅ **Auto-generated documentation** including:
- Version compatibility info
- Installation guide
- Configuration examples

## Installing the Release

1. Download `paimon-trino-478-*.zip` from the release page

2. Extract to Trino plugin directory:
   ```bash
   cd /path/to/trino/plugin
   unzip paimon-trino-478-*.zip
   ```

3. Configure catalog (`/etc/trino/catalog/paimon.properties`):
   ```properties
   connector.name=paimon
   warehouse=/path/to/warehouse
   ```

4. Restart Trino:
   ```bash
   systemctl restart trino
   ```

5. Test connection:
   ```sql
   SHOW CATALOGS;
   SHOW SCHEMAS FROM paimon;
   ```

## Building Different Versions

### For Trino 440
```
Trino version:    440
Airlift version:  [leave empty - auto-detects 241]
Release tag:      v1.0.0-trino-440
Release name:     Paimon Trino 440 Connector v1.0.0
```

### For Trino 468
```
Trino version:    468
Airlift version:  [leave empty - auto-detects 343]
Release tag:      v1.0.0-trino-468
Release name:     Paimon Trino 468 Connector v1.0.0
```

### Custom Airlift Version
```
Trino version:    478
Airlift version:  369
Release tag:      v1.0.0-trino-478-custom
Release name:     Paimon Trino 478 Connector v1.0.0 (Custom)
```

## Manual Build (Alternative)

If you prefer building manually:

```bash
# Clone repository
git clone https://github.com/your-org/paimon-trino.git
cd paimon-trino

# Build
mvn clean package assembly:attached -Papache-build -DskipTests

# Find artifacts
ls -lh target/*.zip target/*.jar
```

## Troubleshooting

### Workflow not visible
- Check: Repository → Settings → Actions → Allow actions

### Build fails
- Review: Actions tab → Click on failed run → Check logs
- Common fix: Re-run the workflow

### Can't download release
- Check: Repository → Settings → Make repository public
- Or: Authenticate with GitHub token

### Plugin not loading in Trino
```bash
# Check plugin directory
ls -la /path/to/trino/plugin/paimon-trino-478/

# Check Trino logs
tail -f /var/log/trino/server.log

# Verify Java version
java -version  # Should be 21+
```

## Quick Links

- 📖 [Full Release Guide](RELEASE_GUIDE.md)
- 🔧 [Workflow Documentation](.github/workflows/README.md)
- 📝 [Changes Summary](CHANGES_SUMMARY.md)
- 🌐 [Trino Documentation](https://trino.io/docs/current/)
- 🌐 [Paimon Documentation](https://paimon.apache.org/)

## Version Matrix

| Trino | Airlift | Java | Status |
|-------|---------|------|--------|
| 478   | 369     | 21   | ✅ Latest |
| 477   | 369     | 21   | ✅ Compatible |
| 476   | 369     | 21   | ✅ Compatible |
| 468   | 343     | 21   | ✅ Compatible |
| 440   | 241     | 21   | ✅ Stable |

## Support

- 🐛 Issues: GitHub Issues tab
- 💬 Discussions: GitHub Discussions
- 📧 Email: [Your contact]
- 📚 Docs: See RELEASE_GUIDE.md

---

**Need Help?** Check [RELEASE_GUIDE.md](RELEASE_GUIDE.md) for detailed information.

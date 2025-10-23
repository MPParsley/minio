# GitHub Actions Workflows

This repository includes automated workflows for building Docker images and syncing with upstream MinIO.

## Docker Build and Push

**File:** `docker-build.yml`

### Purpose
Builds multi-architecture Docker images from source and publishes them to GitHub Container Registry (ghcr.io).

### Features
- **Multi-architecture builds**: amd64, arm64
- **Supply chain security**: SBOM generation, provenance attestations
- **Automatic tagging**: Branch-based, semver, SHA-based tags
- **GitHub Actions caching**: Faster builds with layer caching
- **Non-root container**: Runs as UID 10001 for enhanced security

### Triggers
- Push to `master` or `main` branches
- Push of tags matching `RELEASE.*`
- Pull requests (build only, no push)
- Manual workflow dispatch
- After successful upstream sync

### Usage

#### Pull the Image
```bash
# Latest build from master/main
docker pull ghcr.io/<your-username>/minio:latest

# Specific release
docker pull ghcr.io/<your-username>/minio:RELEASE.2024-01-01T00-00-00Z

# Branch-based
docker pull ghcr.io/<your-username>/minio:master
```

#### Run MinIO
```bash
docker run -p 9000:9000 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=minioadmin \
  -v /data:/data \
  ghcr.io/<your-username>/minio:latest \
  server /data
```

#### Verify Attestations
```bash
# Check provenance
docker buildx imagetools inspect ghcr.io/<your-username>/minio:latest \
  --format "{{json .Provenance}}"

# View SBOM
docker buildx imagetools inspect ghcr.io/<your-username>/minio:latest \
  --format "{{json .SBOM}}"
```

### Configuration

The workflow uses these environment variables:
- `REGISTRY`: ghcr.io
- `IMAGE_NAME`: ${{ github.repository }}

To push to a different registry, modify these variables in the workflow file.

### Security Features

1. **Non-root user**: Container runs as UID 10001
2. **SBOM**: Software Bill of Materials embedded in images
3. **Provenance**: Build attestations for supply chain verification
4. **Minimal base**: Uses Red Hat UBI Micro for minimal attack surface
5. **Build flags**: Uses `-w -s` to strip debugging symbols

## Upstream Sync

**File:** `sync-upstream.yml`

### Purpose
Keeps your fork synchronized with the official MinIO repository.

### Features
- Daily automatic sync at 2 AM UTC
- Manual trigger support
- Fast-forward merge only (safe)
- Syncs tags automatically

### Triggers
- Scheduled: Daily at 2 AM UTC
- Manual workflow dispatch

### Usage

To manually sync:
1. Go to Actions tab
2. Select "Sync Fork with Upstream"
3. Click "Run workflow"

### Safety
The workflow only performs fast-forward merges. If there are conflicts, it will fail and require manual intervention.

## Development

### Building Locally

```bash
# Build for your architecture
docker build -f Dockerfile.build -t minio:local .

# Build multi-arch
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -f Dockerfile.build \
  -t minio:local .
```

### Testing Changes

Create a pull request to test the workflow without pushing images. The workflow will build but not push on PRs.

## Troubleshooting

### Build Fails
- Check the Actions tab for detailed logs
- Verify Dockerfile.build syntax
- Ensure build dependencies are available

### Push Fails
- Verify `GITHUB_TOKEN` has packages:write permission (automatic)
- Check if ghcr.io is accessible
- Ensure image name is lowercase

### Sync Fails
- Check if there are merge conflicts
- Manually sync and resolve conflicts
- Push resolved changes

## Related Documentation

- [GitHub Actions Docker Publishing](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images)
- [Docker Build Push Action](https://github.com/docker/build-push-action)
- [Docker Metadata Action](https://github.com/docker/metadata-action)

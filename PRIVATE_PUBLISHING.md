# Private Publishing Setup (GitHub Packages)

This document describes how to build and release the package and Docker images using GitHub's infrastructure.

## Overview

This fork publishes to GitHub's registries:

- **NPM Package**: Distributed as a **tarball** attached to GitHub Releases.
- **Docker Images**: `ghcr.io/captain-app/sandbox` → GitHub Container Registry

## Prerequisites

No external accounts needed! The workflow uses GitHub's built-in `GITHUB_TOKEN` for:

- Publishing Docker images to GitHub Container Registry
- Creating GitHub releases with tarball assets

## Publishing

### Automatic Publishing

The `release-private.yml` workflow will automatically:

1. Run tests
2. Build packages
3. Create an npm tarball
4. Build and push Docker images to GitHub Container Registry
5. Create a GitHub release with the tarball attached

Triggered on:

- Push to `main` branch
- Manual workflow dispatch

### Manual Publishing

#### NPM Package (Tarball)

```bash
# Build the package
npm run build

# Create tarball
cd packages/sandbox
npm pack
```

#### Docker Images

```bash
# Build and tag images locally
cd packages/sandbox
npm run docker:local

# Login to GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin

# Push images
VERSION=$(node -p "require('./package.json').version")
docker push ghcr.io/captain-app/sandbox:$VERSION
docker push ghcr.io/captain-app/sandbox:$VERSION-python
docker push ghcr.io/captain-app/sandbox:$VERSION-opencode
docker push ghcr.io/captain-app/sandbox:latest
docker push ghcr.io/captain-app/sandbox:latest-python
docker push ghcr.io/captain-app/sandbox:latest-opencode
```

## Using the Published Package

### Install from GitHub Release (Tarball)

The recommended way to use this package is to install it directly from the GitHub Release tarball. This avoids the need for an npm registry or authentication.

In your `package.json`:

```json
{
  "dependencies": {
    "@captain-app/sandbox": "https://github.com/Captain-App/sandbox-sdk/releases/download/vVERSION/sandbox.tgz"
  }
}
```

Replace `VERSION` with the desired version tag (e.g., `0.7.1`).

### Docker Images

```dockerfile
FROM ghcr.io/captain-app/sandbox:latest
# or
FROM ghcr.io/captain-app/sandbox:latest-python
# or
FROM ghcr.io/captain-app/sandbox:latest-opencode
```

**Note**: For private repositories, authenticate to pull images:

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

Or use a GitHub Personal Access Token:

```bash
echo $GITHUB_PAT | docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin
```

## Changes from Upstream

1. **Package Name**: Changed from `@cloudflare/sandbox` to `@cloudflare/sandbox`
2. **Docker Registry**: Changed from `cloudflare/sandbox` to `ghcr.io/captain-app/sandbox`
3. **NPM Registry**: Changed from npmjs.com to GitHub Packages (npm.pkg.github.com)
4. **OpenCode Integration**: Includes the `feature/opencode-integration` branch changes

## Version Management

Versions are managed via package.json. To create a new release:

1. Update version in `packages/sandbox/package.json`
2. Commit and push to `main`
3. The workflow will:
   - Create an npm tarball
   - Build and push Docker images
   - Create a GitHub release with the version tag and tarball asset

Or use changesets (if configured):

```bash
npx changeset
# Select @cloudflare/sandbox and describe changes
git add .changeset
git commit -m "chore: add changeset"
git push
```

## Troubleshooting

### Docker Publishing Fails

- Check that `GITHUB_TOKEN` has `packages:write` permission (automatic in workflows)
- Verify the repository has GitHub Container Registry enabled
- Check image visibility settings (should be private for private repos)

### Can't Pull Docker Images

- Ensure you're authenticated: `docker login ghcr.io`
- For private repos, use a GitHub Personal Access Token with `read:packages` permission
- Check image visibility: https://github.com/Captain-App/sandbox-sdk/pkgs/container/sandbox

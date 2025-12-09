# Private Publishing Setup (GitHub Packages)

This document describes how to publish the npm package and Docker images using GitHub's infrastructure.

## Overview

This fork publishes to GitHub's package registries:
- **NPM Package**: `@captain-app/sandbox` → GitHub Packages (npm registry)
- **Docker Images**: `ghcr.io/captain-app/sandbox` → GitHub Container Registry

## Prerequisites

No external accounts needed! The workflow uses GitHub's built-in `GITHUB_TOKEN` for:
- Publishing npm packages to GitHub Packages
- Publishing Docker images to GitHub Container Registry
- Creating GitHub releases

## Publishing

### Automatic Publishing

The `release-private.yml` workflow will automatically:
1. Run tests
2. Build packages
3. Publish npm package to GitHub Packages
4. Build and push Docker images to GitHub Container Registry
5. Create a GitHub release with tags

Triggered on:
- Push to `main` branch
- Manual workflow dispatch

### Manual Publishing

#### NPM Package (GitHub Packages)

```bash
# Configure npm for GitHub Packages
echo "@captain-app:registry=https://npm.pkg.github.com" >> .npmrc
echo "//npm.pkg.github.com/:_authToken=$GITHUB_TOKEN" >> .npmrc

# Build the package
npm run build

# Publish
cd packages/sandbox
npm publish
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

### Install from GitHub Packages

Configure npm to use GitHub Packages:

**Option 1: Per-project `.npmrc`**
```bash
# In your project root, create/update .npmrc
echo "@captain-app:registry=https://npm.pkg.github.com" >> .npmrc
echo "//npm.pkg.github.com/:_authToken=YOUR_GITHUB_TOKEN" >> .npmrc
```

**Option 2: Global `.npmrc`**
```bash
# In ~/.npmrc
echo "@captain-app:registry=https://npm.pkg.github.com" >> ~/.npmrc
echo "//npm.pkg.github.com/:_authToken=YOUR_GITHUB_TOKEN" >> ~/.npmrc
```

**Option 3: Environment variable**
```bash
export GITHUB_TOKEN=your_token_here
# npm will use GITHUB_TOKEN automatically when configured
```

Then install normally:
```bash
npm install @captain-app/sandbox
```

Or add to `package.json`:
```json
{
  "dependencies": {
    "@captain-app/sandbox": "^0.6.3"
  }
}
```

**Creating a GitHub Token:**
1. Go to https://github.com/settings/tokens
2. Generate new token (classic) with `read:packages` permission
3. Use it in `.npmrc` or as `GITHUB_TOKEN` environment variable

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

1. **Package Name**: Changed from `@cloudflare/sandbox` to `@captain-app/sandbox`
2. **Docker Registry**: Changed from `cloudflare/sandbox` to `ghcr.io/captain-app/sandbox`
3. **NPM Registry**: Changed from npmjs.com to GitHub Packages (npm.pkg.github.com)
4. **OpenCode Integration**: Includes the `feature/opencode-integration` branch changes

## Version Management

Versions are managed via package.json. To create a new release:

1. Update version in `packages/sandbox/package.json`
2. Commit and push to `main`
3. The workflow will:
   - Publish npm package to GitHub Packages
   - Build and push Docker images
   - Create a GitHub release with the version tag

Or use changesets (if configured):
```bash
npx changeset
# Select @captain-app/sandbox and describe changes
git add .changeset
git commit -m "chore: add changeset"
git push
```

## Troubleshooting

### NPM Publishing Fails

- Check that `GITHUB_TOKEN` has `write:packages` permission (automatic in workflows)
- Verify the repository has GitHub Packages enabled
- Check package visibility settings (should be private for private repos)
- View packages at: https://github.com/Captain-App/sandbox-sdk/packages

### Can't Install Package

- Ensure `.npmrc` is configured correctly with GitHub Packages registry
- Verify your GitHub token has `read:packages` permission
- Check package visibility: https://github.com/Captain-App/sandbox-sdk/packages
- Try: `npm install @captain-app/sandbox --verbose` for detailed errors

### Docker Publishing Fails

- Check that `GITHUB_TOKEN` has `packages:write` permission (automatic in workflows)
- Verify the repository has GitHub Container Registry enabled
- Check image visibility settings (should be private for private repos)

### Can't Pull Docker Images

- Ensure you're authenticated: `docker login ghcr.io`
- For private repos, use a GitHub Personal Access Token with `read:packages` permission
- Check image visibility: https://github.com/Captain-App/sandbox-sdk/pkgs/container/sandbox


# AGENTS.md

This file provides guidance to LLMs when working with code in this repository.

## Commands

This project uses Task (Taskfile) for task automation:

- `task` or `task default` - Lists all available tasks
- `task debug` - Shows debugging information including environment variables
- `task install-devbox` - Installs devbox tool (runs once)
- `task docker:build` - Build Docker image with secrets from .env file (with fingerprinting)
- `task docker:build-bake` - Build with Docker Bake (uses docker-bake.hcl configuration, with fingerprinting)
- `task docker:build-dev` - Build development image with .env secrets (with fingerprinting)
- `task docker:build-prod` - Build production image with external secrets (with fingerprinting)
- `task docker:buildx` - Build using docker buildx bake with specified target (with fingerprinting)
- `task docker:up` - Start the project with docker compose
- `task docker:down` - Stop and remove containers, networks, and volumes
- `task docker:logs` - Follow the logs of a running container
- `task docker:exec` - Shell into a running container

## Environment Setup

- Copy `.env.example` to `.env` and configure required environment variables:
  - `REDDIT_APP_NAME`, `REDDIT_APP_CLIENT_ID`, `REDDIT_APP_SECRET` - For Reddit widgets
  - `GITHUB_TOKEN` - For GitHub releases widget
  - `MY_SECRET_TOKEN` - Custom token (example shows 123456)
  - `WORKING_DIR` - Directory for deployment
  - `FQDN` - Fully qualified domain name for deployment

### Docker Build Secrets & Fingerprinting

The project supports secure Docker builds using build secrets and automatic fingerprinting:

- **Build-time secrets**: Available during `RUN` commands but not embedded in final image
- **Environment variables**: Passed as build arguments for non-sensitive values
- **File-based secrets**: `.env` file can be mounted as secret during build
- **Docker Bake**: Supports multiple build targets (dev, prod, platform-specific)
- **Automatic Fingerprinting**: Tasks automatically detect config file changes and rebuild when needed

#### Fingerprinting

Tasks use `sources` and `generates` attributes to fingerprint files:

- **Sources**: `Dockerfile`, `config/**/*`, `assets/**/*`, `.env`
- **Generates**: Docker images with specific tags
- **Benefit**: Tasks only run when source files change, preventing unnecessary rebuilds

Build commands:
```bash
# Direct build with secrets
docker build --secret id=_env,src=.env -t glance:latest .

# Docker Bake (recommended)
docker buildx bake --load

# Build specific targets
docker buildx bake dev --load      # Development build
docker buildx bake prod --load     # Production build
docker buildx bake amd64 --load    # AMD64 specific
docker buildx bake arm64 --load    # ARM64 specific

# Task commands (with automatic fingerprinting)
task docker:build              # Standard build
task docker:build-bake         # Docker Bake build
task docker:build-dev          # Development build
task docker:build-prod         # Production build
task docker:buildx             # Build with specific target
task docker:buildx -- dev      # Development build
task docker:buildx -- prod     # Production build
task docker:buildx -- amd64    # AMD64 platform build
task docker:buildx -- arm64    # ARM64 platform build
```

## Architecture

This is a Glance dashboard configuration repository with deployment automation:

### Configuration Structure

- `config/glance.yml` - Main Glance configuration file that includes other page configs
- `config/home.yml` - Primary dashboard page with widgets (RSS, weather, markets, Reddit, videos, etc.)
- `config/*.yml` - Additional specialized widget configurations (reddit, releases, rss, stock_market, twitch, videos, widgets)

### Deployment

- `compose.yml`/`docker-compose.yml` - Docker Compose configuration for Glance container
- `docker-bake.hcl` - Docker Bake configuration for multi-platform builds with secrets
- Docker builds support secrets via `--secret` flag and `.env` file for secure credential handling

### Widget Configuration

Glance widgets are configured in YAML files using environment variable substitution (e.g., `${GITHUB_TOKEN}`). The main dashboard includes:

- RSS feeds, weather, stock markets, Twitch channels
- YouTube channels via video IDs
- Reddit subreddits with app authentication
- GitHub release tracking

The configuration uses YAML anchors and references (`&reddit-props`, `<<: *reddit-props`) for reusable widget properties.

## Development Guidelines

- Use these tools for linting and formatting
  - editorconfig 
  - `markdownlint -c .markdownlint.jsonc` 
  - ALWAYS add an EOF (trailing newline) regardless of extension
- Test reddit app configuration on remote server via
    ```bash 
    CONTAINER_ID=$(docker ps --filter "name=glance-" --format '{{.ID}}')
    CLIENT_ID=$(docker exec $CONTAINER_ID printenv REDDIT_APP_CLIENT_ID)
    CLIENT_SECRET=$(docker exec $CONTAINER_ID printenv REDDIT_APP_SECRET)
    curl -X POST https://www.reddit.com/api/v1/access_token \
        -H "User-Agent: glance/1.0" \
        -u "$CLIENT_ID:$CLIENT_SECRET" \
        -d "grant_type=client_credentials"
    ```

## Context7 Libraries

- glanceapp/glance
- docker/docs
- stackexchange/dnscontrol
- websites/docs_dokploy_com-docs-core
- websites/taskfile_dev

## Memories

- vps is running debian 12

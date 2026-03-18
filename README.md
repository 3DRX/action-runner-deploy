# GitHub Actions Runner Stack

This stack runs self-hosted GitHub Actions runners with Docker socket access and an optional `redsocks` sidecar for proxying Docker bridge traffic through a host proxy.

## Files

- `docker-compose.yaml` - runner service plus `redsocks` helper services
- `.env.example` - example environment variables
- `redsocks/` - local image and nftables config used by the proxy sidecars

## Prerequisites

- Linux host with Docker Engine and Docker Compose
- A GitHub personal access token that can register org-level self-hosted runners
- A host proxy reachable at `http://172.17.0.1:20172` if you want to use the provided proxy settings
- A writable cache directory at `/srv/github-runner/buildx-cache`

Create the cache directory if needed:

```bash
sudo mkdir -p /srv/github-runner/buildx-cache
sudo chown "$USER":"$USER" /srv/github-runner/buildx-cache
```

## Setup

1. Copy the example env file:

```bash
cp .env.example .env
```

2. Edit `.env` and set a real `ACCESS_TOKEN`.

3. If needed, update `docker-compose.yaml` for your organization and labels:

- `ORG_NAME`
- `RUNNER_NAME_PREFIX`
- `RUNNER_LABELS`

4. If your proxy is different, update these values in `.env`:

- `HTTP_PROXY`
- `HTTPS_PROXY`
- `NO_PROXY`

## Start the stack

Standard start:

```bash
docker compose up -d --build redsocks redsocks_rules gh_runner
```

If your Docker Compose client is newer than the host Docker daemon, use the compatibility workaround:

```bash
DOCKER_API_VERSION=1.41 DOCKER_BUILDKIT=0 COMPOSE_DOCKER_CLI_BUILD=0 docker compose up -d --build redsocks redsocks_rules gh_runner
```

## Scale runners

For plain Docker Compose, use `--scale`:

```bash
docker compose up -d --scale gh_runner=3
```

Note: `deploy.replicas` in `docker-compose.yaml` is only used by Docker Swarm. Plain `docker compose` ignores it.

## Verify

Check service status:

```bash
docker compose ps
```

Watch runner logs:

```bash
docker compose logs -f gh_runner
```

Test Docker Hub access from a bridge container:

```bash
docker run --rm curlimages/curl curl -I https://registry-1.docker.io/v2/
```

## Stop the stack

```bash
docker compose down
```

## Notes

- The real `.env` file is ignored by git; use `.env.example` as the template for new machines.
- The runner uses the host Docker socket, so workflow containers are created by the host Docker daemon.
- The `redsocks` services are included to help bridge-network containers follow the host proxy path.

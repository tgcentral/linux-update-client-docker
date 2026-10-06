# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Docker packaging for No-IP's Linux Dynamic Update Client (`noip-duc`, a Rust binary). The repo holds no application source. The Dockerfile downloads the upstream release tarball (`noip-duc_${VERSION}.tar.gz` from No-IP's CloudFront), builds it with `cargo build --release` in a `rust:*-slim-bookworm` stage, and copies the binary into `debian:bookworm-slim` with `ENTRYPOINT ["/usr/bin/noip-duc"]`.

The container is configured entirely through env vars (`NOIP_USERNAME`, `NOIP_PASSWORD`, `NOIP_HOSTNAMES`, optional `NOIP_IP_METHOD` for IPv6). `noip-duc.env` is an example file with placeholder credentials, and `compose.yaml` loads it.

## Commands

There are no tests or linters.

```bash
# Local build (single arch)
docker build -t noip-duc .
docker build --build-arg VERSION=3.3.0 -t noip-duc .

# Run / inspect options
docker run --rm noip-duc --help
docker run -d --env-file noip-duc.env --name noip-duc noip-duc

# Compose (uses the published ghcr.io image, not the local build)
docker compose up
```

## Release / CI

- `.github/workflows/deploy.yml` runs on **every push to `main`**, on `workflow_dispatch`, and **weekly on a schedule** (Mondays 03:17 UTC) on `ubuntu-latest`. It builds `linux/amd64,linux/arm64,linux/arm/v7` with QEMU/Buildx using `pull: true` and `no-cache: true`, so the runtime stage's `apt-get upgrade` always picks up current Debian security fixes. It **pushes** to `ghcr.io/tgcentral/noip-duc` (this fork) as `:latest`, `:${VERSION}-${PATCH}` (e.g. `3.3.0-p1`) and a dated `:${VERSION}-YYYYMMDD` (UTC build date, e.g. `3.3.0-20261012`). Every build overwrites `:latest` and `:${VERSION}-${PATCH}`. The dated tag is only overwritten by another build on the same UTC day. Docker Hub publishing was removed. Merging to `main` publishes images.
- The workflow does not pass `VERSION` as a build-arg. The tag comes from the workflow's `env.VERSION`, but the binary version comes from the Dockerfile `ARG VERSION` default.

## Bumping the DUC version

The version is hardcoded in three places, and they must stay in sync:
1. `Dockerfile`: `ARG VERSION=` in the builder stage
2. `Dockerfile`: `ARG VERSION=` in the runtime stage (used in the `COPY --from=builder` path)
3. `.github/workflows/deploy.yml`: `env.VERSION` (sets the image tags; the README version badge also reads this value). Reset `env.PATCH` to `p1` on a version bump, and increment it for rebuilds of the same version.

The Rust base image (`rust:1.87.0-slim-bookworm`) may need bumping if the new upstream release needs a newer toolchain.

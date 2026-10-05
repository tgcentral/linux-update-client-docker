# Option A: Patch Debian packages at image build time

## Context
Arcane flags 32 Critical/High CVEs in `noip-duc:3.3.0`. Every one of them is a Debian bookworm OS package (`libssl3`/`openssl`, `libgnutls30`, `libpcre2-8-0`, `libpam*`, `perl-base`, `gpgv`, `libcap2`) in the runtime image, and Debian already ships a fix for each. The published image is just stale. Option A keeps the current Debian runtime and installs the latest bookworm security updates at **image build time**. That produces a new image that scans clean as of the build date.

`apt-get upgrade` runs in the Dockerfile, not inside a running container. If it ran in a running container:
- the fixes would be lost whenever the container is recreated (`docker compose up --force-recreate`, image pull, host reboot with a fresh container);
- the image Arcane scans would still be unchanged;
- the container would be modified by hand, so it would no longer match its image.

## Changes

### 1. `Dockerfile`: runtime stage (lines 21–25)
Replace the existing `RUN apt update ...` block with:
```dockerfile
RUN apt-get update \
 && apt-get upgrade -y \
 && apt-get install -y --no-install-recommends ca-certificates \
 && rm -rf /var/lib/apt/lists/*
```
- `apt-get upgrade -y` pulls every bookworm security update available at build time.
- `apt` becomes `apt-get` (`apt` warns that it has no stable CLI for scripts).
- The builder stage stays as it is. Nothing from it reaches the final image except the binary, and none of the CVEs are in the binary.

### 2. Build so that caching doesn't reuse old layers
Docker reuses a cached layer when the `RUN` line text hasn't changed, so a cached build would silently keep the old packages.
- **Local:** `docker build --pull --no-cache -t noip-duc:3.3.0-p1 .`
  - `--pull` fetches the current `debian:bookworm-slim`.
  - `--no-cache` forces the apt step to re-run.
- **CI (`.github/workflows/deploy.yml`, only if publishing from a fork):** add `pull: true` and `no-cache: true` to the `docker/build-push-action` step. `cache-from: type=gha` would otherwise restore the stale layer.
  - **Cost:** the Rust build also re-runs under QEMU for 3 architectures, which is slow, especially `arm/v7`.

### 3. Tagging
Don't reuse the `3.3.0` tag. Use `3.3.0-p1`, or a date suffix such as `3.3.0-20261005`, so the patched image can be told apart from upstream's.

### 4. `deploy.yml`: required before anything is merged or pushed to the fork's `main`
`deploy.yml` runs on every push to `main`, so update it first:
- **Runner:** change `runs-on: bigger_linux` (No-IP's self-hosted runner) to `ubuntu-latest`. Otherwise the job waits in the queue forever, because no runner with that label exists for the fork.
- **Image tags:** replace the `ghcr.io/noipcom/noip-duc:*` and `noipcom/noip-duc:*` tags with your own (e.g. `ghcr.io/tgcentral/noip-duc:*`). Pushing to `noipcom` would fail.
  - If you drop Docker Hub, remove the `Login to Docker Hub` step too. Otherwise, add `DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN` secrets to the fork.
- Enable Actions on the fork (they are disabled by default on new forks).

## Ongoing upkeep (the main drawback of Option A)
The image is only clean as of its build date. New Debian CVEs will appear, so the image needs rebuilding on a schedule (e.g. a weekly `schedule:` cron trigger in the workflow) or by hand whenever Arcane flags something.

## Verification
1. `docker build --pull --no-cache -t noip-duc:3.3.0-p1 .`
2. Confirm the package versions match or exceed the "Fixed Version" column:
   `docker run --rm --entrypoint dpkg noip-duc:3.3.0-p1 -l | grep -E 'libssl3|openssl|libgnutls30|libpcre2|libpam0g|perl-base|gpgv|libcap2'`
3. Rescan `noip-duc:3.3.0-p1` in Arcane. The expected result is 0 of the 32 listed findings.
4. Check the client still works:
   - `docker run --rm noip-duc:3.3.0-p1 --help`
   - Run it with a real env file, then check `docker logs` for a successful update or "no change" from No-IP.

## Open item
Publishing target: local-only image, or a fork with its own registry tags. This only affects step 2's CI change and the image names in `deploy.yml`.

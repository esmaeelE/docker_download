# docker_dn

Container image downloader — bypass `docker pull` / `podman pull` by downloading image layers directly from the registry.

Based on [Moby's `download-frozen-image-v2.sh`](https://github.com/moby/moby/blob/master/contrib/download-frozen-image-v2.sh) with improvements: `aria2c` parallel downloads, multi-registry support, resume, checksum verification, and colored output.

## Why

When `docker pull` or `podman pull` is blocked (sanctions, 403 from docker.io, corporate proxies), this tool downloads image layers via `aria2c`, creates a tarball, and loads it into Docker or Podman.

```bash
# instead of:
docker pull hello-world:latest
# or
podman pull hello-world:latest

# use:
./docker_dn -d output -i hello-world:latest
```

## Supported Registries

| Registry | Prefix |
|----------|--------|
| Docker Hub | _(default)_ |
| GitHub Container Registry | `ghcr.io/` |
| Google Container Registry | `gcr.io/` |
| Quay | `quay.io/` |

## Install

### Prerequisites

```bash
sudo apt install curl jq aria2c tar gzip
# plus a container runtime for loading (optional with --no-load):
sudo apt install docker.io   # or: sudo apt install podman
```

### Setup

```bash
git clone git@github.com:esmaeelE/docker_download.git
cd docker_download
chmod +x docker_dn
```

Optional — install globally:

```bash
sudo mv docker_dn /usr/local/bin/
```

## Usage

```
docker_dn -d dir -i image[:tag][@digest] ... [options]

Options:
  -d DIR        Output directory for downloading layers
  -i IMAGE      Image name with optional tag/digest (can be repeated)
  -f FILE       Read image list from file (one per line)
  -o FILE       Archive output filename (default: <image>.tar.gz; for multiple
                -i images: <image1>+<image2>.tar.gz)
  --proxy URL   Proxy URL for curl and aria2c (e.g. http://host:port)
  --runtime NAME  Container runtime for loading (docker or podman; default: auto-detect, or CONTAINER_RUNTIME)
  --clean       Remove download directory after successful load
  --no-load     Download and archive only, skip container runtime load
  --dry-run     Show what would be downloaded, then exit
  --verbose     Show debug output
  --quiet       Suppress non-error output
  --version     Show version
  -h, -?        Show this help message
```

When loading is enabled, `docker_dn` auto-detects an available container runtime (`podman` preferred if present, otherwise `docker`) and uses it to load the archive. If the selected runtime is not reachable, it falls back to the other runtime when available (unless `--runtime`/`CONTAINER_RUNTIME` is set). Use `--runtime` or `CONTAINER_RUNTIME` to force a specific runtime, and `--no-load` if no runtime is installed.

### Basic

```bash
./docker_dn -d hello_dir -i hello-world:latest
```

### Force a specific runtime

```bash
./docker_dn -d out -i nginx:alpine --runtime podman
# or
CONTAINER_RUNTIME=docker ./docker_dn -d out -i nginx:alpine
```

### Download only (no container runtime load)

```bash
./docker_dn -d out -i nginx:alpine --no-load
# creates out/nginx.tar.gz
# load later with: docker load -i out/nginx.tar.gz  (or: podman load -i out/nginx.tar.gz)
```

### Custom archive name

```bash
./docker_dn -d out -i redis:7 -o redis-bundle.tar.gz
```

### Multiple images

```bash
./docker_dn -d out -i nginx:alpine -i redis:7 --no-load
# creates out/nginx+redis.tar.gz containing both images,
# loadable with one `docker load` or `podman load`
```

### Batch download from file

```bash
cat images.txt
# hello-world:latest
# nginx:alpine
# postgres:16

./docker_dn -d out -f images.txt --no-load
```

### With proxy

```bash
./docker_dn -d out -i nginx:alpine --proxy http://10.0.0.1:8080
```

### Quiet mode (for scripts)

```bash
./docker_dn -d out -i nginx:alpine --quiet --clean
```

## Features

- **Parallel downloads** — `aria2c` with 16 connections per layer
- **Resume** — interrupted downloads resume from where they stopped (`.done` file tracks completed layers)
- **Checksum verification** — SHA-256 verified against manifest digest after each download
- **Disk space check** — warns if image size exceeds available disk space
- **Multi-registry** — Docker Hub, GHCR, GCR, Quay auto-detected from image prefix
- **Token caching** — one registry token fetched per image, reused across layers, refreshed once on 401
- **Multi-image** — several `-i` images bundled into a single archive with correct per-registry `repositories` keys
- **Parallel multi-image downloads** — when multiple images are requested, image downloads run concurrently and results are aggregated in input order
- **Docker/Podman auto-detect** — auto-selects `podman` or `docker` for loading archives, with fallback to the other runtime if unreachable (or skip with `--no-load`)
- **Colored output** — errors in red, warnings in yellow, success in green (auto-disabled in non-terminal / `--quiet`)
- **Cleanup trap** — partial downloads preserved on failure for resume

## Restricted Countries

If docker.io blocks your IP (403 Forbidden), use the built-in proxy flag:

```bash
./docker_dn -d out -i nginx:alpine --proxy 10.10.34.42:8080
```

Or set proxy via environment:

```bash
export {http,https,ftp}_proxy="10.10.34.42:8080"
./docker_dn -d out -i nginx:alpine
```

Or use `proxychains`:

```bash
proxychains4 ./docker_dn -d out -i nginx:alpine
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Usage error / dependency missing |
| 2 | Auth failure |
| 3 | Download / checksum failure |
| 4 | Container runtime load failure |

## Tested On

- Debian 12, 13
- Ubuntu 24.04

## Links

- [Baeldung post](https://www.baeldung.com/ops/docker-download-image-no-client)
- [Moby's original script](https://github.com/moby/moby/blob/master/contrib/download-frozen-image-v2.sh)

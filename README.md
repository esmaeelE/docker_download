# docker_dn

Download container images directly from registries, build a loadable tar archive, and optionally load it into Docker or Podman — without using `docker pull` or `podman pull`.

Based on [Moby's `download-frozen-image-v2.sh`](https://github.com/moby/moby/blob/master/contrib/download-frozen-image-v2.sh), with support for parallel downloads, multi-image archives, resume/checksum verification, proxying, and Docker/Podman runtime detection.

## Quick start

```bash
git clone git@github.com:esmaeelE/docker_download.git
cd docker_download
chmod +x docker_dn

./docker_dn -d out -i hello-world:latest
```

This downloads `hello-world:latest`, creates an archive in `out/`, and loads it into an available container runtime.

## Requirements

- `bash`, `curl`, `jq`, `aria2c`, `tar`, `gzip`
- `docker` or `podman` **only if loading** (not needed with `--dry-run` or `--no-load`)

Example install (Debian/Ubuntu):

```bash
sudo apt install curl jq aria2c tar gzip
# optional runtime for loading:
sudo apt install podman   # or: sudo apt install docker.io
```

## Usage

```text
docker_dn -d DIR -i IMAGE[:TAG][@DIGEST] ... [options]
```

| Option | Description |
| --- | --- |
| `-d DIR` | Output/download directory (required) |
| `-i IMAGE` | Image reference; repeatable |
| `-f FILE` | Read image references from file (one per line, `#` comments allowed) |
| `-o FILE` | Output archive name (`<image>.tar.gz` by default; `<img1>+<img2>.tar.gz` for multiple images) |
| `--proxy URL` | Proxy URL for `curl` and `aria2c` |
| `--runtime NAME` | Runtime for loading: `docker` or `podman` (default: auto-detect or `CONTAINER_RUNTIME`) |
| `--clean` | Remove download directory after successful load |
| `--no-load` | Download and create archive only |
| `--dry-run` | Show what would be downloaded, then exit |
| `--verbose` | Verbose output |
| `--quiet` | Suppress non-error output |
| `--version` | Show version |
| `-h`, `-?` | Show help |

## Examples

### Download and load

```bash
./docker_dn -d out -i nginx:alpine
```

### Download only (no runtime load)

```bash
./docker_dn -d out -i nginx:alpine --no-load
# archive: out/nginx.tar.gz
# load later:
#   docker load -i out/nginx.tar.gz
#   podman load -i out/nginx.tar.gz
```

### Multiple images in one archive

```bash
./docker_dn -d out -i nginx:alpine -i redis:7 --no-load
# archive: out/nginx+redis.tar.gz
```

When multiple images are requested, they are downloaded in parallel and bundled into a single archive with results kept in input order.

### Batch from file

```bash
cat > images.txt <<'EOF'
hello-world:latest
ghcr.io/owner/app:1.2.3
postgres:16
EOF

./docker_dn -d out -f images.txt --no-load
```

### Force a runtime

```bash
./docker_dn -d out -i nginx:alpine --runtime podman
# or
CONTAINER_RUNTIME=docker ./docker_dn -d out -i nginx:alpine
```

### Custom archive name / proxy / quiet cleanup

```bash
./docker_dn -d out -i redis:7 -o redis-bundle.tar.gz
./docker_dn -d out -i nginx:alpine --proxy http://10.0.0.1:8080
./docker_dn -d out -i nginx:alpine --quiet --clean
```

## Supported registries

| Registry | Prefix | Example |
| --- | --- | --- |
| Docker Hub | _(default)_ | `hello-world:latest`, `library/alpine:3.20` |
| GitHub Container Registry | `ghcr.io/` | `ghcr.io/owner/image:tag` |
| Google Container Registry | `gcr.io/` | `gcr.io/project/image:tag` |
| Quay | `quay.io/` | `quay.io/org/image:tag` |

## Behavior notes

- **Runtime selection:** auto-detects `podman` first, then `docker`. If auto-detected runtime is unreachable, `docker_dn` may fall back to the other runtime; explicit `--runtime` / `CONTAINER_RUNTIME` disables fallback.
- **Resume:** rerun with the same `-d` directory to continue interrupted downloads (completed blobs tracked in `.done`).
- **Integrity:** downloaded blobs are verified against manifest SHA-256 digests.
- **Concurrency safety:** the download directory is locked to prevent simultaneous writes by multiple instances.
- **Archive naming:** single image -> `<image>.tar.gz`; multiple images -> `<image1>+<image2>.tar.gz` (slashes normalized), unless `-o` is set.
- **Cleanup/failure:** partial downloads are preserved on failure for resume; `--clean` removes the download directory after a successful load flow.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Success |
| `1` | Usage or dependency error |
| `2` | Authentication failure |
| `3` | Download or checksum failure |
| `4` | Container runtime load failure |

## Links

- [Moby: `download-frozen-image-v2.sh`](https://github.com/moby/moby/blob/master/contrib/download-frozen-image-v2.sh)
- [Baeldung: Download Docker image without Docker client](https://www.baeldung.com/ops/docker-download-image-no-client)

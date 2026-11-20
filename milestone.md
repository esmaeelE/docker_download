# Milestones

## High Priority (Quick Wins)

### 1. Fix filename bug — DONE
Archive naming now avoids `/` issues by normalizing image names (e.g. `${image//\//_}`) and supports multi-image archive names (e.g. `a+b.tar.gz`).

### 2. Quote all variables — IN PROGRESS (improved)
Most command/path handling is quoted. ShellCheck (`-S warning`) is clean and style-level issues found were fixed (e.g. direct exit-status checks for `aria2c`/runtime load); remaining style finding is a false positive on the trap-installed cleanup function. Further auditing for uncommon paths/inputs can continue.

### 3. Check dependencies/startup requirements — DONE (updated)
Startup checks core tools (`curl`, `jq`, `aria2c`, `tar`, `gzip`). Container runtime handling now supports Docker or Podman:
- auto-detects `podman` (preferred) or `docker` for loading
- allows explicit selection via `--runtime` / `CONTAINER_RUNTIME`
- requires a runtime only when loading is needed (`--dry-run`/`--no-load` can run without one)

### 4. Proper CLI flags — DONE
The script uses flag-based CLI (`-d`, `-i`, `-f`, `-o`, `--proxy`, `--clean`, `--no-load`, `--dry-run`, etc.) with `-h`/`--version` support.

---

## Medium Priority

### 5. Multi-registry support — DONE
Supported registries include Docker Hub (`docker.io`, default), GHCR (`ghcr.io`), GCR (`gcr.io`), and Quay (`quay.io`) with prefix-based detection and registry-specific auth endpoints.

### 6. Resume on failure — DONE
Completed blobs/layers are tracked in `$dir/.done` so interrupted downloads can be resumed by rerunning the same command.

### 7. Cleanup on failure — DONE (resume-friendly)
Failure handling preserves partial downloads when a download is interrupted/incomplete so they can be resumed; successful runs clean up state as appropriate (and `--clean` removes the download directory after a successful load flow).

---

## Low Priority

### 8. Consider rewriting in Python — OPEN
The Bash script remains featureful but complex (HTTP/auth flows, manifests, JSON/state handling). A Python rewrite could improve maintainability, argument parsing, and error handling.

### 9. Batch download support — DONE
Images can be provided via `-f <file>` (one image per line) in addition to repeated `-i` arguments.

### 10. Checksum verification — DONE
Downloaded blobs are verified against manifest digests (SHA-256) before being recorded as complete.

### 11. Parallel multi-image download — DONE
When multiple images are requested, per-image processing runs in parallel background jobs with aggregation in original argument order. Shared blob downloads are synchronized with per-digest locks to handle shared layers safely.

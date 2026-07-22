# Milestones

## High Priority (Quick Wins)

### 1. Fix filename bug
`${image}.tar.gz` breaks when `$image` contains `/` (e.g., `library/hello-world`):
```bash
imageFile="${image//\//_}"  # use existing var, already defined in script
tar -cvC "${dir}" . | gzip -c -9 >"${imageFile}.tar.gz"
```

### 2. Quote all variables
Unquoted variables break on paths with spaces:
```bash
# Bad:
dn $blobRedirect $targetFile
# Good:
dn "$blobRedirect" "$targetFile"
```

### 3. Check all dependencies at startup
Currently only checks `curl` and `jq`, missing `aria2c`, `tar`, `gzip`, `docker`:
```bash
for cmd in curl jq aria2c tar gzip docker; do
    if ! command -v "$cmd" &>/dev/null; then
        echo >&2 "error: \"$cmd\" not found!"
        exit 1
    fi
done
```

### 4. Proper CLI flags
Replace positional args with flags:
```bash
./docker_dn -d output_dir -i hello-world:latest
./docker_dn -h  # help
```

---

## Medium Priority

### 5. Multi-registry support
Add support for other registries:

| Registry | Base URL |
|----------|----------|
| GCR | `gcr.io` |
| GHCR | `ghcr.io` |
| Quay | `quay.io` |

Detection logic: parse image prefix → set registry base URL.

### 6. Resume on failure
Track completed layers in a state file. On restart, skip completed layers.

### 7. Cleanup on failure
Trap errors and remove partial downloads:
```bash
cleanup() { rm -rf "$dir"; }
trap cleanup ERR
```

---

## Low Priority

### 8. Consider rewriting in Python
The Bash script is already ~480 lines with complex logic (HTTP redirects, JSON parsing, state management). Python would:
- Better error handling
- Easier multi-registry support
- Proper argument parsing with `argparse`
- `requests` library handles redirects/headers cleanly
- Cross-platform without bash version hacks

### 9. Batch download support
Read a list of images from a file:
```bash
./docker_dn -f images.txt  # downloads all images in file
```

### 10. Checksum verification
Verify downloaded layer integrity against manifest digests.

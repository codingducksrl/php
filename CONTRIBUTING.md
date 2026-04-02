# Contributing

## Repository structure

```
.
├── prod/
│   └── laravel/
│       ├── php84/
│       │   ├── Dockerfile              # PHP 8.4 production image
│       │   ├── docker-php-entrypoint  # Custom entrypoint script
│       │   └── config.json            # Version + node versions config
│       └── php85/
│           ├── Dockerfile              # PHP 8.5 production image
│           ├── docker-php-entrypoint  # Custom entrypoint script
│           └── config.json            # Version + node versions config
└── .github/
    └── workflows/
        ├── build-prod.yml             # CI: orchestrates build and merge
        ├── _build.yml                 # Reusable: builds arch-specific images
        └── _merge.yml                 # Reusable: merges into multi-arch manifests
```

The `dev/` directory is reserved for future development images.

---

## CI/CD

### `build-prod.yml`

Builds and pushes all production images to GHCR.

**Triggers:**
- Push to `main` when any file under `prod/` changes.
- Manual dispatch via the GitHub Actions UI (`workflow_dispatch`).

**Matrix:** `php_version × node_version × arch` — dynamically built from each `config.json`, currently 12 parallel build jobs (2 PHP versions × 3 Node versions × 2 architectures), followed by 6 manifest-merge jobs. Builds run with `fail-fast: false`, so a single failure does not cancel the others.

**Multi-arch strategy:** amd64 and arm64 are built natively on separate runners (`ubuntu-latest` and `ubuntu-24-arm`) to avoid slow QEMU emulation. Each build job pushes an arch-specific intermediate tag (e.g. `php8.4-node22-amd64`). Once both arches are pushed, a `merge` job combines them into a single multi-arch manifest list using `docker buildx imagetools create`.

**Permissions required:** the workflow uses `GITHUB_TOKEN` with `packages: write` — no secrets need to be configured manually.

**Layer caching:** GitHub Actions cache is used, scoped per `php_dir + node_version + arch`. Subsequent runs only rebuild layers that have changed.

---

## Versioning

Each PHP version directory contains a `config.json`:

```json
{
  "version": "1.0.0",
  "node_versions": [20, 22, 24],
  "default_node": 24,
  "is_latest": true
}
```

- **`version`** — read by CI to publish pinned tags (e.g. `php8.5-node24-1.0.0`, `php8.5-1.0.0`). Bump this and push to `main` to release a new image version.
- **`node_versions`** — which Node.js versions to build for this PHP release.
- **`default_node`** — the Node version used for the PHP alias tag (e.g. `php8.5`).
- **`is_latest`** — if `true`, this PHP version also receives the `latest` tag (only one version should have this set).

A build is only triggered for a PHP version if its `config.json` version string changed compared to `HEAD~1`. `workflow_dispatch` always builds all versions.

---

## Adding a new PHP version

1. Create the directory and files:
   ```
   prod/laravel/php86/
   ├── Dockerfile
   ├── docker-php-entrypoint
   └── config.json
   ```
   Copy from an existing version and adjust the PHP version number.

2. Write `config.json` for the new version:
   ```json
   {
     "version": "1.0.0",
     "node_versions": [20, 22, 24],
     "default_node": 24,
     "is_latest": true
   }
   ```

3. If this version should become `latest`, set `"is_latest": false` in the previous version's `config.json`.

No changes to the CI pipeline are needed — the workflow auto-discovers all `prod/laravel/php*/config.json` files.

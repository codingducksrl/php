# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Docker image repository for production-ready Laravel applications. Images are built on [FrankenPHP](https://frankenphp.dev/) with PHP (8.4/8.5) and Node.js (20/22/24), published to GHCR as `ghcr.io/codingducksrl/laravel-php`.

There is no local build or test command — all builds run through GitHub Actions (`.github/workflows/build-prod.yml`). To trigger a build, push to `main` with changes under `prod/`, or use `workflow_dispatch` from the GitHub Actions UI.

## Repository structure

```
prod/laravel/
  php84/   Dockerfile + docker-php-entrypoint  (PHP 8.4 image)
  php85/   Dockerfile + docker-php-entrypoint  (PHP 8.5 image)
.github/workflows/build-prod.yml              CI pipeline
```

The `dev/` directory is reserved for future development images and does not yet exist.

## CI/CD architecture

The workflow builds a `php_version × node_version × arch` matrix — 12 parallel build jobs (2 PHP × 3 Node × 2 arch), then 6 manifest-merge jobs. amd64 and arm64 are built natively on separate runners (`ubuntu-latest` / `ubuntu-24-arm`) to avoid QEMU. Arch-specific tags (e.g. `php8.4-node22-amd64`) are merged into multi-arch manifests with `docker buildx imagetools create`. No secrets need configuring; the workflow uses `GITHUB_TOKEN` with `packages: write`.

## Adding a new PHP version

1. Create `prod/laravel/php<XY>/Dockerfile` and `docker-php-entrypoint` (copy from an existing version and adjust).
2. In `.github/workflows/build-prod.yml`, add the new version to the `php_version` array and an `include` block (with `php_dir` and `default_node`) in **both** the `build` and `merge` jobs.
3. Update the `latest` tag condition in the `Compute final tags` step of the `merge` job to point to the new version.

## Entrypoint behaviour

`docker-php-entrypoint` runs every executable file in `/init.d/` (if it exists) before handing off. If the first argument starts with `-`, it prepends `frankenphp run`. Mount scripts at `/init.d/` for startup hooks (migrations, cache warming) without modifying the image.

## Image tag convention

- `php8.4-node20` / `php8.4-node22` / … — specific combination (recommended for production)
- `php8.4` / `php8.5` — PHP alias → default Node (22 for 8.4, 24 for 8.5)
- `latest` — newest PHP at its default Node (currently PHP 8.5, Node 24)

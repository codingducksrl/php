# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Docker image repository for Laravel applications with two image families:

- **`laravel-php`** — production images built on [FrankenPHP](https://frankenphp.dev/) with PHP (8.4/8.5) and Node.js (20/22/24), published to GHCR as `ghcr.io/codingducksrl/laravel-php`.
- **`laravel-sail`** — development images (Ubuntu 24.04 + supervisord, matching Laravel Sail) with PHP (8.4/8.5) and Node.js (22/24), published to GHCR as `ghcr.io/codingducksrl/laravel-sail`.

There is no local build or test command — all builds run through GitHub Actions (`.github/workflows/build-prod.yml`). To trigger a build, push to `main` with changes under `prod/`, or use `workflow_dispatch` from the GitHub Actions UI.

## Repository structure

```
prod/laravel/
  php84/   Dockerfile + docker-php-entrypoint + config.json  (PHP 8.4 FrankenPHP image)
  php85/   Dockerfile + docker-php-entrypoint + config.json  (PHP 8.5 FrankenPHP image)
prod/sail/
  php84/   Dockerfile + start-container + supervisord.conf + php.ini + config.json  (PHP 8.4 Sail image)
  php85/   Dockerfile + start-container + supervisord.conf + php.ini + config.json  (PHP 8.5 Sail image)
.github/workflows/build-prod.yml                             CI pipeline
```

The `dev/` directory is reserved for future development images and does not yet exist.

## CI/CD architecture

The `detect` job auto-discovers all image types by globbing `prod/*/php*/config.json` and builds an explicit matrix from each file's `node_versions`, `default_node`, and `is_latest` fields. The image name (`laravel-php` or `laravel-sail`) is derived from the directory name (`prod/laravel` → `laravel-php`, `prod/sail` → `laravel-sail`). amd64 and arm64 are built natively on separate runners (`ubuntu-latest` / `ubuntu-24-arm`) to avoid QEMU. Arch-specific tags (e.g. `php8.4-node22-amd64`) are merged into multi-arch manifests with `docker buildx imagetools create`. No secrets need configuring; the workflow uses `GITHUB_TOKEN` with `packages: write`.

A build is only triggered for an image if its `config.json` version string changed (compared to `HEAD~1`). `workflow_dispatch` always builds all versions.

## Adding a new PHP version

The steps are the same for both image families — just use the appropriate `prod/<type>/` directory.

1. Create `prod/laravel/php<XY>/Dockerfile` and `docker-php-entrypoint` (copy from an existing version and adjust), or `prod/sail/php<XY>/` with its files.
2. Create `config.json` with the version string, node versions to build, default node, and whether this is the `latest` alias:
   ```json
   {
     "version": "1.0.0",
     "node_versions": [22, 24],
     "default_node": 24,
     "is_latest": true
   }
   ```
3. If this version should become `latest`, set `"is_latest": false` in the previous version's `config.json`.

The workflow auto-discovers all `prod/*/php*/config.json` files — no changes to the CI pipeline are needed.

## Entrypoint behaviour

**laravel-php:** `docker-php-entrypoint` runs every executable file in `/init.d/` (if it exists) before handing off. If the first argument starts with `-`, it prepends `frankenphp run`. Mount scripts at `/init.d/` for startup hooks (migrations, cache warming) without modifying the image.

**laravel-sail:** `start-container` launches supervisord, which runs PHP via `artisan serve`. The `WWWGROUP` build arg must be passed at build time (typically `$(id -g)` from docker-compose) to set the `sail` user's group ID.

## Image tag convention

Tags are the same for both image families (applied to their respective GHCR packages):

- `php8.4-node22` / `php8.4-node24` / … — specific combination (recommended for production)
- `php8.4` / `php8.5` — PHP alias → default Node (22 for 8.4, 24 for 8.5)
- `latest` — newest PHP at its default Node (currently PHP 8.5, Node 24)

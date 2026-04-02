# PHP Docker Images

Production-ready Docker images for Laravel applications, built on [FrankenPHP](https://frankenphp.dev/) with Node.js support. Images are published to the GitHub Container Registry (GHCR) as multi-arch manifests supporting `linux/amd64` and `linux/arm64`.

---

## Images

### Available images

| Image | PHP | Node | Tag type |
|-------|-----|------|----------|
| `ghcr.io/codingducksrl/laravel-php:php8.4-node20` | 8.4 | 20 | specific |
| `ghcr.io/codingducksrl/laravel-php:php8.4-node22` | 8.4 | 22 | specific |
| `ghcr.io/codingducksrl/laravel-php:php8.4-node24` | 8.4 | 24 | specific |
| `ghcr.io/codingducksrl/laravel-php:php8.4` | 8.4 | 22 | PHP alias |
| `ghcr.io/codingducksrl/laravel-php:php8.5-node20` | 8.5 | 20 | specific |
| `ghcr.io/codingducksrl/laravel-php:php8.5-node22` | 8.5 | 22 | specific |
| `ghcr.io/codingducksrl/laravel-php:php8.5-node24` | 8.5 | 24 | specific |
| `ghcr.io/codingducksrl/laravel-php:php8.5` | 8.5 | 24 | PHP alias |
| `ghcr.io/codingducksrl/laravel-php:latest` | 8.5 | 24 | latest stable |

### Tag convention

Tags follow the pattern `php<PHP_VERSION>-node<NODE_VERSION>`.

- **Specific tags** (`php8.4-node20`) — pin to an exact combination. Recommended for production deployments.
- **PHP alias** (`php8.4`, `php8.5`) — points to the default Node version for that PHP release (22 for PHP 8.4, 24 for PHP 8.5).
- **`latest`** — always points to the newest PHP release at its default Node version (currently PHP 8.5, Node 24).

---

## What's included

Each image ships with:

| Component | Details |
|-----------|---------|
| [FrankenPHP](https://frankenphp.dev/) | latest for the PHP release |
| PHP | 8.4 or 8.5 |
| Composer | 2 |
| Node.js | 20 / 22 / 24 (matrix) |
| ffmpeg | system latest |

### PHP extensions

`bcmath`, `exif`, `gd`, `igbinary`, `imagick`, `intl`, `mysqli`, `opcache`, `pdo`, `pdo_mysql`, `pdo_pgsql`, `pgsql`, `redis`, `sockets`, `zip`, `pcntl`

### Runtime defaults

- **`SERVER_NAME=:80`** — FrankenPHP listens on port 80.
- **Production php.ini** — `php.ini-production` is used, not the development preset.
- System cron jobs are removed to keep the image clean.

---

## Repository structure

```
.
├── prod/
│   └── laravel/
│       ├── php84/
│       │   ├── Dockerfile              # PHP 8.4 production image
│       │   └── docker-php-entrypoint  # Custom entrypoint script
│       └── php85/
│           ├── Dockerfile              # PHP 8.5 production image
│           └── docker-php-entrypoint  # Custom entrypoint script
└── .github/
    └── workflows/
        └── build-prod.yml             # CI: builds and pushes prod images
```

The `dev/` directory is reserved for future development images.

---

## Entrypoint

The custom entrypoint (`docker-php-entrypoint`) does two things before handing off to the main process:

1. **Init scripts** — if `/init.d/` exists in the container, every executable file inside it is run in order. Use this to run migrations, warm caches, or perform any startup work without modifying the image.
2. **FrankenPHP fallback** — if the first argument starts with `-`, it prepends `frankenphp run` automatically (matches the upstream FrankenPHP convention).

To hook into container startup, mount a directory of scripts at `/init.d/` and make them executable.

---

## CI/CD

### `build-prod.yml`

Builds and pushes all production images to GHCR.

**Triggers:**
- Push to `main` when any file under `prod/` changes.
- Manual dispatch via the GitHub Actions UI (`workflow_dispatch`).

**Matrix:** `php_version × node_version × arch` → 12 parallel build jobs (2 PHP versions × 3 Node versions × 2 architectures), followed by 6 manifest-merge jobs. Builds run with `fail-fast: false`, so a single failure does not cancel the others.

**Multi-arch strategy:** amd64 and arm64 are built natively on separate runners (`ubuntu-latest` and `ubuntu-24-arm`) to avoid slow QEMU emulation. Each build job pushes an arch-specific intermediate tag (e.g. `php8.4-node22-amd64`). Once both arches are pushed, a `merge` job combines them into a single multi-arch manifest list using `docker buildx imagetools create`.

**Permissions required:** the workflow uses `GITHUB_TOKEN` with `packages: write` — no secrets need to be configured manually.

**Layer caching:** GitHub Actions cache is used, scoped per `php_dir + node_version + arch`. Subsequent runs only rebuild layers that have changed.

---

## Adding a new PHP version

1. Create the directory and files:
   ```
   prod/laravel/php86/
   ├── Dockerfile
   └── docker-php-entrypoint
   ```

2. Add an entry to the `include` block in **both** the `build` and `merge` jobs in `.github/workflows/build-prod.yml`:
   ```yaml
   matrix:
     php_version: ["8.4", "8.5", "8.6"]   # add version here
     node_version: [20, 22, 24]
     runner: [ubuntu-latest, ubuntu-24-arm]  # build job only
     include:
       - php_version: "8.4"
         php_dir: php84
         default_node: 22
       - php_version: "8.5"
         php_dir: php85
         default_node: 24
       - php_version: "8.6"               # add this block
         php_dir: php86
         default_node: 24
   ```

3. Update the `latest` tag condition in the `Compute final tags` step (in the `merge` job) to point to the new version.

The new version will be built for all three Node variants automatically.

---

## Pulling an image

```sh
# Latest stable (PHP 8.5, Node 24)
docker pull ghcr.io/codingducksrl/laravel-php:latest

# Pin to a specific PHP + Node combination
docker pull ghcr.io/codingducksrl/laravel-php:php8.4-node22
docker pull ghcr.io/codingducksrl/laravel-php:php8.5-node20
```

## Using as a base image

```dockerfile
FROM ghcr.io/codingducksrl/laravel-php:php8.5-node24

WORKDIR /app
COPY . .
RUN composer install --no-dev --optimize-autoloader
```

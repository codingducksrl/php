# PHP Docker Images

Production-ready Docker images for Laravel applications, built on [FrankenPHP](https://frankenphp.dev/) with Node.js support. Images are published to the GitHub Container Registry (GHCR) as multi-arch manifests supporting `linux/amd64` and `linux/arm64`.

---

## Pulling an image

```sh
# Latest stable (PHP 8.5, Node 24)
docker pull ghcr.io/codingducksrl/laravel-php:latest

# Pin to a specific PHP + Node combination
docker pull ghcr.io/codingducksrl/laravel-php:php8.4-node22
docker pull ghcr.io/codingducksrl/laravel-php:php8.5-node20

# Pin to a specific image version (recommended for production)
docker pull ghcr.io/codingducksrl/laravel-php:php8.5-node24-1.0.0
```

## Using as a base image

```dockerfile
FROM ghcr.io/codingducksrl/laravel-php:php8.5-node24

WORKDIR /app
COPY . .
RUN composer install --no-dev --optimize-autoloader
```

---

## Available images

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

- **Specific tags** (`php8.4-node20`) — pin to an exact PHP + Node combination.
- **Versioned tags** (`php8.4-node22-1.0.0`, `php8.4-1.0.0`) — pin to a specific image version. Recommended for production: the image will never change under you, so updates are always explicit.
- **PHP alias** (`php8.4`, `php8.5`) — points to the default Node version for that PHP release (22 for PHP 8.4, 24 for PHP 8.5).
- **`latest`** — always points to the newest PHP release at its default Node version (currently PHP 8.5, Node 24).

> **Production recommendation:** use a versioned tag (e.g. `php8.5-node24-1.0.0`) in your Dockerfile. Mutable tags like `latest` or `php8.5` can change when a new image is published, which may cause unexpected breakage in automated workflows.

---

## What's included

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

## Entrypoint

The custom entrypoint runs every executable file in `/init.d/` before starting the main process. Use this to run migrations, warm caches, or perform any startup work without modifying the image.

```sh
# Mount a directory of scripts at /init.d/ and make them executable
docker run --volume ./startup:/init.d ghcr.io/codingducksrl/laravel-php:latest
```

---

For information on the repository structure, CI/CD pipeline, and how to add a new PHP version, see [CONTRIBUTING.md](CONTRIBUTING.md).

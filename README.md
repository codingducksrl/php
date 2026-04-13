# PHP Docker Images

Docker images for Laravel applications published to the GitHub Container Registry (GHCR) as multi-arch manifests supporting `linux/amd64` and `linux/arm64`. Two image families are available:

- **`laravel-php`** — production images built on [FrankenPHP](https://frankenphp.dev/)
- **`laravel-sail`** — development images matching [Laravel Sail](https://laravel.com/docs/sail) (Ubuntu 24.04 + supervisord)

---

## laravel-php

Production-ready images built on FrankenPHP.

### Pulling

```sh
# Latest stable (PHP 8.5, Node 24)
docker pull ghcr.io/codingducksrl/laravel-php:latest

# Pin to a specific PHP + Node combination
docker pull ghcr.io/codingducksrl/laravel-php:php8.4-node22
docker pull ghcr.io/codingducksrl/laravel-php:php8.5-node24

# Pin to a specific image version (recommended for production)
docker pull ghcr.io/codingducksrl/laravel-php:php8.5-node24-1.0.0
```

### Using as a base image

```dockerfile
FROM ghcr.io/codingducksrl/laravel-php:php8.5-node24

WORKDIR /app
COPY . .
RUN composer install --no-dev --optimize-autoloader
```

### Available tags

| Tag | PHP | Node | Type |
|-----|-----|------|------|
| `php8.3-node20` | 8.3 | 20 | specific |
| `php8.3-node22` | 8.3 | 22 | specific |
| `php8.3` | 8.3 | 22 | PHP alias |
| `php8.4-node22` | 8.4 | 22 | specific |
| `php8.4-node24` | 8.4 | 24 | specific |
| `php8.4` | 8.4 | 22 | PHP alias |
| `php8.5-node22` | 8.5 | 22 | specific |
| `php8.5-node24` | 8.5 | 24 | specific |
| `php8.5` | 8.5 | 24 | PHP alias |
| `latest` | 8.5 | 24 | latest stable |

### What's included

| Component | Details |
|-----------|---------|
| [FrankenPHP](https://frankenphp.dev/) | latest for the PHP release |
| PHP | 8.3, 8.4, or 8.5 |
| Composer | 2 |
| Node.js | 20 / 22 / 24 |
| ffmpeg | system latest |

PHP extensions: `bcmath`, `exif`, `gd`, `igbinary`, `imagick`, `intl`, `mysqli`, `opcache`, `pdo`, `pdo_mysql`, `pdo_pgsql`, `pgsql`, `redis`, `sockets`, `zip`, `pcntl`

Runtime defaults: FrankenPHP listens on port 80 (`SERVER_NAME=:80`), `php.ini-production` preset, cron removed.

### Entrypoint

Runs every executable file in `/init.d/` before starting the main process. Use this to run migrations, warm caches, or perform any startup work without modifying the image.

```sh
docker run --volume ./startup:/init.d ghcr.io/codingducksrl/laravel-php:latest
```

---

## laravel-sail

Development images matching the Laravel Sail environment. Based on Ubuntu 24.04 with supervisord managing the PHP process via `artisan serve`.

### Pulling

```sh
# Latest stable (PHP 8.5, Node 24)
docker pull ghcr.io/codingducksrl/laravel-sail:latest

# Pin to a specific PHP + Node combination
docker pull ghcr.io/codingducksrl/laravel-sail:php8.4-node22
docker pull ghcr.io/codingducksrl/laravel-sail:php8.5-node24

# Pin to a specific image version
docker pull ghcr.io/codingducksrl/laravel-sail:php8.5-node24-1.0.0
```

### Available tags

| Tag | PHP | Node | Type |
|-----|-----|------|------|
| `php8.4-node22` | 8.4 | 22 | specific |
| `php8.4-node24` | 8.4 | 24 | specific |
| `php8.4` | 8.4 | 22 | PHP alias |
| `php8.5-node22` | 8.5 | 22 | specific |
| `php8.5-node24` | 8.5 | 24 | specific |
| `php8.5` | 8.5 | 24 | PHP alias |
| `latest` | 8.5 | 24 | latest stable |

### What's included

| Component | Details |
|-----------|---------|
| Base | Ubuntu 24.04 |
| PHP | 8.4 or 8.5 (ondrej/php PPA) |
| Composer | 2 |
| Node.js | 22 / 24 |
| Process manager | supervisord |
| ffmpeg | system latest |

PHP extensions: `bcmath`, `gd`, `igbinary`, `imagick`, `intl`, `imap`, `ldap`, `mbstring`, `memcached`, `msgpack`, `mysql`, `pcov`, `pgsql`, `readline`, `redis`, `soap`, `sqlite3`, `xdebug`, `xml`, `zip`

> **Note:** the `WWWGROUP` build arg must be passed at build time to set the `sail` user's group ID — typically done by docker-compose with `--build-arg WWWGROUP=$(id -g)`.

---

## Tag convention (both families)

- **Specific tags** (`php8.5-node24`) — pin to an exact PHP + Node combination.
- **Versioned tags** (`php8.5-node24-1.0.0`, `php8.5-1.0.0`) — pin to a specific image version. The image will never change under you, so updates are always explicit.
- **PHP alias** (`php8.4`, `php8.5`) — points to the default Node version for that PHP release.
- **`latest`** — always points to the newest PHP release at its default Node version (currently PHP 8.5, Node 24).

> **Production recommendation:** use a versioned tag in your Dockerfile. Mutable tags like `latest` or `php8.5` can change when a new image is published.

---

For information on the repository structure, CI/CD pipeline, and how to add a new PHP version, see [CONTRIBUTING.md](CONTRIBUTING.md).

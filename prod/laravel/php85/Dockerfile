FROM dunglas/frankenphp:php8.5

ARG NODE_VERSION=24

RUN cp $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

RUN apt-get -yq update \
    && apt-get install -yq --no-install-recommends \
        ffmpeg \
        libfcgi-bin \
        cron \
    && curl -fL https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -yq --no-install-recommends nodejs \
    && apt-get clean \
    && apt-get autoremove -y \
    && rm -rf /var/cache/apt/* /var/lib/apt/lists/* \
    # remove all the cron jobs shipped with the system
    && rm -f /etc/cron.{d,daily,hourly,monthly,weekly,yearly}/[!.]*

RUN install-php-extensions \
        bcmath \
        exif \
        gd \
        igbinary \
        imagick \
        intl \
        mysqli \
        opcache \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        pgsql \
        redis \
        sockets \
        zip \
        pcntl

ARG DEFAULT_SERVER_NAME=:80
ENV SERVER_NAME=${DEFAULT_SERVER_NAME}

EXPOSE 80

COPY ./docker-php-entrypoint /usr/local/bin/docker-php-entrypoint
RUN chmod +x /usr/local/bin/docker-php-entrypoint

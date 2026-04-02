FROM dunglas/frankenphp:php8.4

ARG NODE_VERSION=22

RUN cp $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

RUN \
    apt-get -yq update && apt-get install -yq \
        ffmpeg \
        libfcgi-bin \
        cron

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

RUN \
    apt-get -yq clean \
        && apt-get -yq autoremove \
        && rm -rdf /var/cache/apt/* /var/lib/apt/lists/* \
    # remove all the cron jobs shipped with the system
    && rm -f /etc/cron.{d,daily,hourly,monthly,weekly,yearly}/[!.]*

RUN curl -sLS https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs

ENV SERVER_NAME=:80

COPY ./docker-php-entrypoint /usr/local/bin/docker-php-entrypoint
RUN chmod +x /usr/local/bin/docker-php-entrypoint

FROM serversideup/php:8.5-frankenphp

ARG NODE_VERSION=24

WORKDIR /app

ENV DEBIAN_FRONTEND=noninteractive

USER root

RUN echo "Acquire::http::Pipeline-Depth 0;" > /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::http::No-Cache true;" >> /etc/apt/apt.conf.d/99custom && \
    echo "Acquire::BrokenProxy    true;" >> /etc/apt/apt.conf.d/99custom

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
    msgpack \
    pgsql \
    sockets

#ARG DEFAULT_SERVER_NAME=:80
#ENV SERVER_NAME=${DEFAULT_SERVER_NAME}

USER www-data

EXPOSE 8000/tcp

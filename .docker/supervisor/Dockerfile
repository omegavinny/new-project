ARG PHP_VERSION=8.4

FROM php:${PHP_VERSION}-alpine

LABEL maintainer="Marcus Costa <marcus.alfieri@outlook.com>"

USER root

RUN apk --update add wget \
    curl \
    git \
    build-base \
    icu-dev \
    libzip-dev \
    libmcrypt-dev \
    libxml2-dev \
    linux-headers \
    mysql-client \
    mariadb-connector-c \
    pcre-dev \
    postgresql-dev \
    zlib-dev \
    autoconf \
    cyrus-sasl-dev \
    libgsasl-dev \
    oniguruma-dev \
    openssl-dev \
    supervisor

RUN pecl channel-update pecl.php.net \
    && pecl install -o -f redis \
    && pecl install xlswriter

RUN docker-php-ext-configure intl

RUN docker-php-ext-install bcmath intl mysqli mbstring pdo pdo_mysql pdo_pgsql xml pcntl zip

RUN docker-php-ext-enable intl redis xlswriter

# Install GD
RUN apk add --update --no-cache freetype-dev libjpeg-turbo-dev jpeg-dev libpng-dev \
    && docker-php-ext-configure gd \
    && docker-php-ext-install gd

# Install imagick
RUN apk add --update --no-cache imagemagick-dev imagemagick \
    && cd /tmp \
    && git clone https://github.com/Imagick/imagick \
    && cd imagick \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && rm -r /tmp/imagick \
    && docker-php-ext-enable imagick

# Configura Supervisor
COPY supervisord.conf /etc/supervisor/
ENTRYPOINT  ["/usr/bin/supervisord", "-n", "-c",  "/etc/supervisor/supervisord.conf"]

# Configura o timezone
ARG TIMEZONE=UTC
ENV TIMEZONE ${TIMEZONE}
RUN ln -snf /usr/share/zoneinfo/$TIMEZONE /etc/localtime && echo $TIMEZONE > /etc/timezone

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}

ARG PGID=1000
ENV PGID ${PGID}

ARG USERNAME=default
ENV USERNAME ${USERNAME}

RUN addgroup -g ${PGID} ${USERNAME} && \
    adduser -D -G ${USERNAME} -u ${PUID} ${USERNAME}

# Clean up
RUN rm /var/cache/apk/* \
    && mkdir -p /var/www

WORKDIR /etc/supervisor

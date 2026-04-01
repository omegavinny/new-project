ARG PHP_VERSION=8.4
FROM php:${PHP_VERSION}-fpm

LABEL maintainer="Marcus Costa <marcus.alfieri@outlook.com>"

ENV DEBIAN_FRONTEND=noninteractive

USER root

RUN set -xe; \
    apt-get update -yqq \
    && pecl channel-update pecl.php.net \
    && apt-get install -yqq \
    apt-utils \
    bash \
    ca-certificates \
    curl \
    gnupg2 \
    git \
    libpq-dev \
    libzip-dev \
    nano \
    postgresql-client \
    unzip \
    zip \
    && docker-php-ext-install bcmath opcache pgsql pdo_pgsql

# Copy opcache
COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini
COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

# Install decimal extension
RUN apt-get install -yqq libmpdec-dev \
    && pecl install decimal \
    docker-php-ext-enable decimal

# Install gd extension
RUN apt-get install -yqq libfreetype-dev libjpeg62-turbo-dev libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

# Install imagick extension
RUN apt-get install -yqq libmagickwand-dev imagemagick \
    && cd /tmp \
    && git clone https://github.com/Imagick/imagick \
    && cd imagick \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && rm -r /tmp/imagick \
    && docker-php-ext-enable imagick

# Install intl extension
RUN apt-get install -yqq zlib1g-dev libicu-dev g++ \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl

# Install redis extension
RUN pecl install -o -f redis \
    && docker-php-ext-enable redis

# Install xsl extension
RUN apt-get -y install libxslt-dev \
    && docker-php-ext-install xsl

# Install xlswriter extension
RUN pecl install xlswriter \
    && docker-php-ext-enable xlswriter

# Install zip extension
RUN docker-php-ext-configure zip \
    && docker-php-ext-install zip

# Configura o timezone
ARG TIMEZONE=UTC
ENV TIMEZONE ${TIMEZONE}

RUN ln -snf /usr/share/zoneinfo/$TIMEZONE /etc/localtime && echo $TIMEZONE > /etc/timezone

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}

ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -o -g ${PGID} www-data \
    && usermod -o -u ${PUID} -g www-data www-data \
    && usermod -d /home/www-data -m -s /bin/bash www-data \
    && chown -Rf www-data:www-data /home/www-data

# Configure locale.
ARG LOCALE=POSIX
ENV LC_ALL ${LOCALE}

# Clean up
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && rm -f /var/log/lastlog /var/log/faillog

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000

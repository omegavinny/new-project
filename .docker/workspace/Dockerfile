ARG PHP_VERSION=8.4

FROM php:${PHP_VERSION}-cli

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

ARG USERNAME=default
ENV USERNAME ${USERNAME}

# Criação direta do grupo e do usuário com as configurações fornecidas
RUN groupadd -g ${PGID} ${USERNAME} \
    && useradd -u ${PUID} -g ${PGID} -m -s /bin/bash ${USERNAME} \
    && chown -Rf ${USERNAME}:${USERNAME} /home/${USERNAME}

# Users Aliases
COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/${USERNAME}/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/${USERNAME}/aliases.sh && \
    chown ${USERNAME}:${USERNAME} /home/${USERNAME}/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

# Instala o Composer
RUN curl -sS https://getcomposer.org/installer -o composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm composer-setup.php

# Add the composer.json
COPY ./composer.json /home/${USERNAME}/.composer/composer.json
RUN chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.composer

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

# Update composer
ARG COMPOSER_VERSION=2
ENV COMPOSER_VERSION ${COMPOSER_VERSION}

# Update composer
ARG COMPOSER_VERSION=2
ENV COMPOSER_VERSION ${COMPOSER_VERSION}

RUN set -eux; \
    if [ "$COMPOSER_VERSION" = "1" ] || [ "$COMPOSER_VERSION" = "2" ] || [ "$COMPOSER_VERSION" = "2.2" ]; then \
    composer self-update --${COMPOSER_VERSION}; \
    else \
    composer self-update ${COMPOSER_VERSION}; \
    fi

# Crontab
RUN mkdir -p /etc/cron.d \
    && echo "* * * * * ${USERNAME} /usr/bin/php /var/www/artisan schedule:run >> /dev/null 2>&1" >> /etc/cron.d/${USERNAME} \
    && chmod -R 644 /etc/cron.d

COPY ./ca-certificates/* /usr/local/share/ca-certificates/
RUN update-ca-certificates

USER ${USERNAME}

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc

# Export composer vendor path
RUN composer global install \
    && composer global require "laravel/installer"

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

# Install: NVM + Node + NPM
ENV NVM_DIR /home/${USERNAME}/.nvm

ARG NODE_VERSION=24
ENV NODE_VERSION ${NODE_VERSION}

RUN mkdir -p $NVM_DIR \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash \
    && . $NVM_DIR/nvm.sh \
    && nvm install ${NODE_VERSION} \
    && nvm use ${NODE_VERSION} \
    && nvm alias ${NODE_VERSION} \
    && npm cache clear --force \
    && npm config set fetch-retries 2 \
    && npm config set fetch-retry-factor 10 \
    && npm config set fetch-retry-mintimeout 10000 \
    && npm config set fetch-retry-maxtimeout 60000

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm -f /var/log/lastlog /var/log/faillog \
    && rm -rf /tmp/pear

# Set default work directory
WORKDIR /var/www

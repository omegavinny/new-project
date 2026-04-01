FROM nginx:alpine

LABEL mainteiner="Marcus Costa <marcus.alfieri@outlook.com>"

COPY nginx.conf /etc/nginx/

RUN apk update \
    && apk upgrade \
    && apk --update add logrotate \
    && apk add --no-cache bash curl openssl

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=82
ENV PUID ${PUID}
ARG PGID=82
ENV PGID ${PGID}

RUN set -x ; \
    addgroup -g ${PGID} -S www-data ; \
    adduser -u ${PUID} -D -S -G www-data www-data && exit 0 ; exit 1

# Create 'messages' file used from 'logrotate'
RUN touch /var/log/messages

# Copy 'logrotate' config file
COPY logrotate/nginx /etc/logrotate.d/

# Set upstream conf and remove the default conf
ARG PHP_UPSTREAM_CONTAINER=php-fpm
ENV PHP_UPSTREAM_CONTAINER ${PHP_UPSTREAM_CONTAINER}

ARG PHP_UPSTREAM_PORT=9000
ENV PHP_UPSTREAM_PORT ${PHP_UPSTREAM_PORT}

RUN echo "upstream php-upstream { server ${PHP_UPSTREAM_CONTAINER}:${PHP_UPSTREAM_PORT}; }" > /etc/nginx/conf.d/upstream.conf \
    && rm /etc/nginx/conf.d/default.conf

ADD ./startup.sh /opt/startup.sh
RUN sed -i 's/\r//g' /opt/startup.sh
CMD ["/bin/bash", "/opt/startup.sh"]

EXPOSE 80 81 443

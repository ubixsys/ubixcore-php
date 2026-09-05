# ubixcore-php 8.5 — the uBixCore PHP runtime: nginx + PHP-FPM 8.5 + memcache(d)
# on Alpine, supervised, running as an unprivileged user, listening on 8080.
#
# Alpine is pinned: a floating tag once moved PHP under our feet. Bump the pin
# and the phpNN package family together, on purpose, in their own commit.
FROM alpine:3.23
LABEL org.opencontainers.image.title="ubixcore-php" \
      org.opencontainers.image.description="uBixCore PHP 8.5 runtime: nginx + PHP-FPM + memcache(d) on Alpine" \
      org.opencontainers.image.source="https://github.com/ubixsys/ubixcore-php" \
      org.opencontainers.image.licenses="BSD-3-Clause" \
      org.opencontainers.image.vendor="uBixSys"

RUN apk --no-cache add \
    curl \
    nginx \
    php85 \
    php85-ctype \
    php85-curl \
    php85-dom \
    php85-fileinfo \
    php85-fpm \
    php85-gd \
    php85-iconv \
    php85-intl \
    php85-json \
    php85-mbstring \
    php85-mysqli \
    php85-opcache \
    php85-openssl \
    php85-pdo \
    php85-pdo_mysql \
    php85-pecl-memcache \
    php85-pecl-memcached \
    php85-phar \
    php85-session \
    php85-simplexml \
    php85-tokenizer \
    php85-xml \
    php85-xmlreader \
    php85-xmlwriter \
    php85-zip \
    php85-zlib \
    supervisor \
    tzdata \
    ca-certificates \
    && rm -rf /var/cache/apk/* /tmp/* \
    && ln -sf /usr/bin/php85 /usr/bin/php \
    && php85 -r "copy('https://getcomposer.org/installer', '/tmp/composer-setup.php');" \
    && php85 /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm -f /tmp/composer-setup.php

# nginx, PHP-FPM, PHP, supervisord
COPY config/nginx.conf /etc/nginx/nginx.conf
ENV PHP_INI_DIR=/etc/php85
COPY config/fpm-pool.conf ${PHP_INI_DIR}/php-fpm.d/www.conf
COPY config/php.ini      ${PHP_INI_DIR}/conf.d/zzz_custom.ini
COPY config/php-fpm.ini  ${PHP_INI_DIR}/conf.d/yyy_fpm.ini
COPY config/limits.conf  /etc/security/limits.conf
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Logs to the container's stdout/stderr
RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log

# Unprivileged runtime user; /web/root is the document root your image COPYs into
RUN adduser -D -g 'www' www \
    && chown -R www:www /run /var/lib/nginx /var/tmp/nginx /var/log/nginx \
    && mkdir -p /web/root /logs && chown -R www:www /web /logs

EXPOSE 8080
HEALTHCHECK --timeout=10s CMD curl --silent --fail http://127.0.0.1:8080/fpm-ping
USER www
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

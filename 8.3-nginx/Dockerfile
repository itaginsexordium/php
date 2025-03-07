#  +----------------------------------------------------------+
#  | Main image                                               |
#  +----------------------------------------------------------+
FROM alptraum/s6-overlay:v3.1.5.0 as main

ARG WAIT_FOR_VERSION=v2.2.4 \
    PGID=9999 \
    PUID=9999 \
    WEBUSER_HOME="/var/www/html"



ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    S6_CMD_WAIT_FOR_SERVICES_MAXTIME=0 \
    PHP_DATE_TIMEZONE="UTC" \
    PHP_DISPLAY_ERRORS=Off \
    PHP_DISPLAY_STARTUP_ERRORS=Off \
    PHP_ERROR_REPORTING="22527" \
    PHP_MEMORY_LIMIT="256M" \
    PHP_MAX_EXECUTION_TIME="99" \
    PHP_OPEN_BASEDIR="${WEBUSER_HOME}:/dev/stdout:/tmp:/var/www" \
    PHP_POST_MAX_SIZE="100M" \
    PHP_UPLOAD_MAX_FILE_SIZE="100M" \
    PHP_POOL_NAME="www" \
    PHP_PM_CONTROL=dynamic \
    PHP_PM_MAX_CHILDREN="20" \
    PHP_PM_START_SERVERS="2" \
    PHP_PM_MIN_SPARE_SERVERS="1" \
    PHP_PM_MAX_SPARE_SERVERS="3"

RUN apk add -U --no-cache \
    # Install deps
    curl \
    supercronic \
    nginx \
    fcgi \
    tzdata \
    shadow \
    # Install php and extensions
    php83 \
    php83-fpm \
    php83-opcache \
    php83-ctype \
    php83-curl \
    php83-iconv \
    php83-xmlreader \
    php83-xmlwriter \
    php83-simplexml \
    php83-dom \
    php83-fileinfo \
    php83-mbstring \
    php83-openssl \
    php83-pdo \
    php83-pdo_sqlite \
    php83-sqlite3 \
    php83-pdo_pgsql \
    php83-pgsql \
    php83-redis \
    php83-session \
    php83-tokenizer \
    php83-xml \
    php83-phar \
    # Install `php-fpm-healthcheck` script
    && curl -o /usr/local/bin/php-fpm-healthcheck https://raw.githubusercontent.com/renatomefi/php-fpm-healthcheck/master/php-fpm-healthcheck \
    && chmod +x /usr/local/bin/php-fpm-healthcheck \
    # Install 'wait-for' script
    && curl -o /usr/local/bin/wait-for https://raw.githubusercontent.com/eficode/wait-for/${WAIT_FOR_VERSION}/wait-for \
    && chmod +x /usr/local/bin/wait-for \
    # Allow `php-fpm` script to be referenced without a version
    && ln -s /usr/sbin/php-fpm83 /usr/sbin/php-fpm \
    # Configure web user and group
    && addgroup --system --gid ${PGID} webgroup \
    && adduser --system --home ${WEBUSER_HOME} --uid ${PUID} --ingroup webgroup webuser \
    # Implement changes required to run s6-overlay as an unprivileged user
    && chown -R webuser /etc/s6-overlay \
    && chmod -R g+w /etc/s6-overlay \
    # Implement changes required to run php-fpm as an unprivileged user
    && chown -R webuser /etc/php83 \
    && chmod -R g+w /etc/php83 \
    && chown -R webuser /var/log/php83 \
    && chmod -R g+w /var/log/php83 \
    # Forward request and error nginx logs to docker log collector
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log \
    # Create a docker-entrypoint.d directory
    && mkdir /docker-entrypoint.d \
    # Implement changes required to run NGINX as an unprivileged user
    # Nginx user must own the cache and etc directory to write cache and tweak the nginx config
    && mkdir /var/cache/nginx \
    && chown -R webuser /var/cache/nginx \
    && chmod -R g+w /var/cache/nginx \
    && chown -R webuser /etc/nginx \
    && chmod -R g+w /etc/nginx \
    && chown -R webuser /var/log/nginx \
    && chmod -R g+w /var/log/nginx \
    && chown -R webuser /var/lib/nginx \
    && chmod -R g+w /var/lib/nginx \
    # Cleanup
    && rm -rf /etc/nginx/conf.d/* /etc/php83/php-fpm.d/* /var/www/html/*

COPY --chmod=755 rootfs/etc /etc
COPY --chown=webuser:webgroup www-root /var/www

USER webuser
WORKDIR ${WEBUSER_HOME}
ENTRYPOINT ["/init"]
EXPOSE 80

#  +----------------------------------------------------------+
#  | Dev image                                                |
#  +----------------------------------------------------------+


FROM node:23.5-alpine AS node

FROM main as dev

ARG COMPOSER_VERSION=2.5.7

USER root

COPY --from=node /usr/lib /usr/lib
COPY --from=node /usr/local/lib /usr/local/lib
COPY --from=node /usr/local/include /usr/local/include
COPY --from=node /usr/local/bin /usr/local/bin

RUN apk -U upgrade && apk add --no-cache \
    yarn \
    # php83-xdebug \
    # Download composer.
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=${COMPOSER_VERSION} 

COPY --chmod=755 devfs/etc /etc

USER webuser

EXPOSE 80 9003
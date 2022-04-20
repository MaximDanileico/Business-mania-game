FROM php:8.1-fpm-alpine

RUN pecl channel-update pecl.php.net && \
    apk update && \
    apk upgrade && \
    apk add --no-cache --no-progress git openssh curl zsh zsh-vcs && \
    mkdir -p /tmp && \
    mkdir -p /scripts

RUN set -eux; \
    apk add --no-cache --virtual .build-deps autoconf make g++ \
                                                            libzip-dev \
                                                            libxml2-dev \
                                                            freetype-dev \
                                                            libpng-dev \
                                                            libwebp-dev \
                                                            libjpeg-turbo-dev \
                                                            libxpm-dev \
                                                            icu-dev && \
    ### PHP EXTENSION - ZIP
    apk add --no-cache --no-progress libzip && \
    docker-php-ext-configure zip --with-zip && \
    docker-php-ext-install zip && \
    ### PHP EXTENSION - SOAP
    apk add --no-cache --no-progress libxml2 && \
    docker-php-ext-install soap && \
    ### PHP EXTENSION - MYSQLI
    docker-php-ext-install mysqli && \
    ### PHP EXTENSION - PDO_MYSQL
    docker-php-ext-configure pdo_mysql --with-zlib-dir=/usr && \
    docker-php-ext-install pdo_mysql && \
    ### PHP EXTENSION - CURL
    apk add --no-cache --no-progress freetype libpng libwebp libjpeg-turbo libxpm && \
    ln -s /usr/lib/x86_64-linux-gnu/libXpm.* /usr/lib/ && \
    docker-php-ext-configure gd --enable-gd --with-webp --with-jpeg --with-xpm --with-freetype --enable-gd-jis-conv && \
    docker-php-ext-install gd && \
    ### PHP EXTENSION - OPCACHE
    docker-php-ext-install opcache && \
    docker-php-ext-enable opcache && \
    ### PHP EXTENSION - XDEBUG
    pecl install xdebug-3.1.1 && \
    docker-php-ext-enable xdebug && \
    ### PHP EXTENSION - SOCKETS
    docker-php-ext-configure sockets && \
    docker-php-ext-install sockets && \
    ### PHP EXTENSION - INTL
    apk add --no-cachphpe --no-progress icu && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl && \
    ### PHP EXTENSION - BCMATH
    docker-php-ext-install bcmath && \
    ### END INSTALL
    # Clean up build packages
    docker-php-source delete && \
    apk del .build-deps && \
    rm -rf /usr/local/etc/php-fpm.d/zz-docker.conf && \
    rm -rf /usr/local/etc/php-fpm.d/docker.conf && \
    true

ARG USER_NAME="www-data"
ENV USER_NAME ${USER_NAME}
ARG USER_ID=1000
ENV USER_ID ${USER_ID}
ARG GROUP_NAME="www-data"
ENV GROUP_NAME ${GROUP_NAME}
ARG GROUP_ID=1000
ENV GROUP_ID ${GROUP_ID}

RUN apk add --no-cache --virtual .build-deps shadow && \
    usermod --uid ${USER_ID} ${USER_NAME} && \
    groupmod --gid ${GROUP_ID} ${GROUP_NAME} && \
    apk del .build-deps && \
    true

RUN set -eux; \
    mkdir -p /root/.ssh && \
    chmod 0700 /root/.ssh && \
    ssh-keyscan bitbucket.org >> /root/.ssh/known_hosts && \
    printf "Host bitbucket.org\nIdentityFile /root/.ssh/id_rsa\n" > /root/.ssh/config && \
    chmod 600 /root/.ssh/config && \
    cp -R /root/.ssh /home/"${USER_NAME}"/ && \
    chown -R "${USER_ID}":"${GROUP_ID}" /home/"${USER_NAME}" && \
    true

COPY ./modules/php/configs/conf.d/* /usr/local/etc/php/conf.d/
COPY ./modules/php/configs/php-fpm.d/* /usr/local/etc/php-fpm.d/
COPY ./modules/php/configs/php-fpm.conf /usr/local/etc/php-fpm.conf
COPY ./modules/php/configs/php.ini /usr/local/etc/php/php.ini

RUN set -eux \
    chmod +x "$(php -r 'echo ini_get("extension_dir");')"/* && \
    (find /usr/local/bin -type f -print0 | xargs -n1 -0 strip --strip-all -p 2>/dev/null || true) && \
    (find /usr/local/lib -type f -print0 | xargs -n1 -0 strip --strip-all -p 2>/dev/null || true) && \
    (find /usr/local/sbin -type f -print0 | xargs -n1 -0 strip --strip-all -p 2>/dev/null || true) && \
    true

ARG COMPOSER_VERSION=2.1.9
ENV COMPOSER_MEMORY_LIMIT=-1
RUN set -eux; \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer --version=${COMPOSER_VERSION} && \
    sh -c '[[ $COMPOSER_VERSION == "1"* ]] && composer global require hirak/prestissimo && rm -rf /home/$USER_NAME/.composer/cache || true' && \
    cp -R /root/.composer /home/"${USER_NAME}"/ && \
    chown -R ${USER_NAME} /home/${USER_NAME}/.composer

RUN rm -rf /var/cache/apk && \
    rm -rf /root/.composer/cache && \
    rm -rf /tmp/* && \
    rm -rf /var/www/html && \
    chown "${USER_ID}":"${GROUP_ID}" /var/www

STOPSIGNAL SIGQUIT

EXPOSE 9000
CMD ["php-fpm"]

# Metadata
LABEL org.opencontainers.image.vendor="Maxim Danileico" \
    org.opencontainers.image.title="PHP-FPM v8.1 Alpine" \
    org.opencontainers.image.description="PHP-FPM v8.1 with essential extensions on top of Alpine Linux." \
    org.opencontainers.image.version="8.1"
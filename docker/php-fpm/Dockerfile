#######################################
# Build
#######################################
FROM php:7.4-fpm-buster AS build

# Arguments defined in docker-compose.yml
ARG user
ARG group
ARG uid
ARG composer_user

WORKDIR /application

# libpng-dev needed by "gd" extension
# libzip-dev needed by "zip" extension
# libicu-dev for intl extension
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    wget \
    gnupg \
    g++ \
    locales \
    unzip \
    dialog \
    apt-utils \
    curl \
    git \
    bash \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    libicu-dev \
    vim \
    && apt-get clean

# Create system user to run Composer
RUN useradd -G www-data,root -u $uid -d /home/$composer_user $composer_user
RUN mkdir -p /home/$composer_user/.composer && \
    chown -R $composer_user:$composer_user /home/$composer_user

COPY ./docker/php-fpm/composer-installer.sh /usr/local/bin/composer-installer
RUN apt-get -yqq update \
    && chmod +x /usr/local/bin/composer-installer \
    && composer-installer \
    && mv composer.phar /usr/local/bin/composer \
    && chmod +x /usr/local/bin/composer \
    && composer --version \
    && chown -R $group:$user /var/www/ \
    && chown -R $group:$user ~/.composer/

# install necessary tools for running application
# install opcache, apcu, pdo_mysql
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install \
    gd \
    intl \
    opcache \
    apcu \
    zip \
    pdo_mysql \
    && docker-php-source extract \
    && pecl install opcache apcu \
    && docker-php-ext-enable \
    opcache \
    apcu \
    pdo_mysql \
    && docker-php-source delete \
    && echo "apc.enable_cli=1\n" >> "$PHP_INI_DIR/conf.d/docker-php-ext-apcu.ini" \
    && rm -rf /tmp/*

# set UTC server time
RUN echo "UTC" > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata

# allow webserver to to use modify application
RUN chown -R $group:$user /application/

#######################################
# Testing
#######################################
FROM build AS testing

RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"

RUN pecl install xdebug \
    && echo "xdebug.remote_enable=on\n" >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini" \
    && echo "xdebug.remote_autostart=on\n" >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini" \
    && echo "xdebug.remote_port=9000\n" >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini" \
    && echo "xdebug.remote_handler=dbgp\n" >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini" \
    && echo "xdebug.remote_connect_back=1\n" >> "$PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini" \
    && docker-php-ext-enable xdebug \
    && rm -rf /tmp/*

USER $user

#######################################
# Standard
#######################################
FROM build AS standard

COPY ./app /application
RUN composer install --no-dev --no-interaction
ENV PATH="~/.composer/vendor/bin:./vendor/bin:${PATH}"

USER $user
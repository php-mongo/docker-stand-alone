# Dockerfile
FROM php:7.4.26-apache-buster

MAINTAINER Gilbert Rehling <gilbert@phpmongoadmin.com>

ENV PHPMONGOADMIN_STAND_ALONE=1

ARG WITH_XDEBUG=false
ARG WITH_XDEBUG_PROFILE=false

# Basic system requirements
RUN apt-get update -y \
    && apt-get install -y \
      autoconf \
      coreutils \
      gcc \
      g++ \
      git \
      nmap \
      make \
      openssl \
      pkg-config \
      unzip \
      vim \
      nano

# System dev requirements
RUN apt-get update \
    && apt-get install -y \
        libjpeg-dev \
        libpng-dev \
        libfreetype6-dev \
        libpq-dev \
        libcurl4-openssl-dev \
        libssl-dev \
        libmcrypt-dev

# PHP modules
# Intl
RUN apt-get update \
    && apt-get install -y libicu-dev \
    && docker-php-ext-configure intl \
    && docker-php-ext-install -j$(nproc) \
        intl

# xml
RUN apt-get update \
    && apt-get install -y \
        libxml2-dev \
        libxslt-dev \
        && docker-php-ext-install -j$(nproc) \
          dom \
          xmlrpc \
          xml \
          xsl

# database
RUN apt-get update \
    && apt-get install -y \
        sqlite3 \
        libsqlite3-dev \
        && docker-php-ext-install -j$(nproc) \
            mysqli \
            pdo \
            pdo_sqlite

# strings
RUN apt-get update \
    && apt-get install -y libonig-dev \
    && docker-php-ext-install -j$(nproc) \
        gettext \
        mbstring

# math
RUN apt-get update \
    && apt-get install -y libgmp-dev \
    && ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h \
    && docker-php-ext-install -j$(nproc) \
        gmp \
        bcmath

# compression
RUN apt-get update \
    && apt-get install -y \
        libbz2-dev \
        zlib1g-dev \
        libzip-dev \
        && docker-php-ext-install -j$(nproc) \
          zip \
          bz2

# ssh2
RUN apt-get update \
    && apt-get install -y \
        libssh2-1-dev

# memcached
RUN apt-get update \
    && apt-get install -y \
        libmemcached-dev \
        libmemcached11

# the rest
RUN docker-php-ext-install -j$(nproc) \
    soap \
    sockets \
    calendar \
    ctype \
    exif \
    gd \
    fileinfo \
    iconv \
    intl \
    json \
    tokenizer \
    opcache

# PECL installs
RUN pecl install mongodb
RUN docker-php-ext-enable mongodb;
RUN pecl install mcrypt
RUN docker-php-ext-enable mcrypt;
RUN pecl install memcached
RUN docker-php-ext-enable memcached;

# Optional: redis for planned caching improvements
#RUN pecl install -D 'enable-redis-igbinary="no" enable-redis-lxf="no" enable-redis-zstd="no"' redis
#RUN docker-php-ext-enable redis;

# Install XDebug, but not enable by default. Enable using:
# * php -d$XDEBUG_EXT vendor/bin/phpunit
# * php_xdebug vendor/bin/phpunit
RUN pecl install xdebug
ENV XDEBUG_EXT zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20190902/xdebug.so
RUN alias php_xdebug="php -d $XDEBUG_EXT vendor/bin/phpunit"

# Install composer - taken from composer dockerfile
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /tmp
COPY --from=composer:1.10 /usr/bin/composer /usr/bin/composer
# Install composer and put binary into $PATH
#RUN curl -sS https://getcomposer.org/installer | php \
#	&& mv composer.phar /usr/local/bin/ \
#	&& ln -s /usr/local/bin/composer.phar /usr/local/bin/composer

# Copy the php.ini file
COPY config/php.ini-development /usr/local/etc/php.ini

# Copy the web host configuration
RUN a2enmod rewrite
COPY config/phpMongoAdmin.conf /etc/apache2/conf-available/
RUN ln -s /etc/apache2/conf-available/phpMongoAdmin.conf /etc/apache2/conf-enabled/phpMongoAdmin.conf

# Copy default index file for apache directory
# COPY ../../../var/www/html/index.html /var/www/html/index.html

# Set the working directory to make it easier to enter
WORKDIR /usr/share/phpMongoAdmin

# Give the www-data user ownership of the web volume
RUN chown -R www-data:www-data /var/www

# Copy any user scripts to the container
RUN mkdir -p /tmp/scripts
COPY ./scripts/*.sh /tmp/scripts/

ENV BASH_ENV /etc/profile

# Add bash helpers and extras. A bit of magic to glue everything together
# Add your favourite aliases etc...
RUN touch /tmp/scripts/profile.sh \
    && echo "alias lsa='ls -la'" >> /tmp/scripts/profile.sh \
    && chmod +x /tmp/scripts/* \
    && cp /tmp/scripts/profile.sh /etc/profile.d/ \
    && cp /tmp/scripts/phpmongoadmin.sh /etc/profile.d/ \
    && echo ". /etc/profile" >> ~/.bashrc

# Cleanup
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /var/cache/*



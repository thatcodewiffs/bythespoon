FROM php:8.0.2-fpm
LABEL maintainer="Cohen Harvey <cohen.n.harvey@gmail.com>"

ARG XDEBUG_HOST="host.docker.internal"
ARG XDEBUG_CONF="disabled"
ARG ENV_STATE="production"

ENV LANGUAGE   en_US.UTF-8
ENV LANG       en_US.UTF-8
ENV LC_ALL     en_US.UTF-8

# Set working directory
WORKDIR /var/www/html

# Ensure UTF-8
RUN DEBIAN_FRONTEND="noninteractive" apt-get clean && apt-get update
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y apt-utils locales

RUN echo "LC_ALL=en_US.UTF-8" >> /etc/environment
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "LANG=en_US.UTF-8" > /etc/locale.
RUN DEBIAN_FRONTEND="noninteractive" locale-gen en_US.UTF-8

RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y vim curl wget build-essential git

# Install node
RUN DEBIAN_FRONTEND="noninteractive" curl -sL https://deb.nodesource.com/setup_16.x | bash -
RUN DEBIAN_FRONTEND="noninteractive" apt-get install -y nodejs

# Install system dependencies
RUN DEBIAN_FRONTEND="noninteractive" apt-get update && apt-get install -y \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libpq-dev \
    libicu-dev \
    libssl-dev \
    jpegoptim optipng pngquant gifsicle \
    zip \
    unzip \
    libzip-dev \
    libgtk2.0-0 \
    libgtk-3-0 \
    libgbm-dev \
    libnotify-dev \
    libgconf-2-4 \
    libnss3 \
    libxss1 \
    libasound2 \
    libxtst6 \
    xauth \
    xvfb

# Install PHP extensions
RUN docker-php-ext-install -j$(nproc) opcache bcmath zip pdo_mysql pcntl gd
RUN docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/

# Install XDEBUG
RUN pecl install xdebug
RUN mkdir -p /usr/local/etc/php/${XDEBUG_CONF}
RUN \
    echo "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20200930/xdebug.so" > /usr/local/etc/php/${XDEBUG_CONF}/xdebug.ini && \
    echo "xdebug.mode = debug" >> /usr/local/etc/php/${XDEBUG_CONF}/xdebug.ini && \
    echo "xdebug.start_with_request = yes" >> /usr/local/etc/php/${XDEBUG_CONF}/xdebug.ini && \
    echo "xdebug.discover_client_host = 0" >> /usr/local/etc/php/${XDEBUG_CONF}/xdebug.ini && \
    echo "xdebug.client_port = 9003" >> /usr/local/etc/php/${XDEBUG_CONF}/xdebug.ini && \
    echo "xdebug.client_host =" ${XDEBUG_HOST} >> /usr/local/etc/php/${XDEBUG_CONF}/xdebug.ini && \
    echo "xdebug.output_dir=\"/tmp\"" >> /usr/local/etc/php/${XDEBUG_CONF}/xdebug.ini

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

##############################################################################
# phpunit
##############################################################################

ENV PHPUNIT_VERSION 9.5.20
RUN mkdir -p /root/src \
    && cd /root/src \
    && wget https://phar.phpunit.de/phpunit-${PHPUNIT_VERSION}.phar \
    && chmod +x phpunit-${PHPUNIT_VERSION}.phar \
    && mv phpunit-${PHPUNIT_VERSION}.phar /usr/local/bin/phpunit \
    && rm -rf /root/src \
    && phpunit --version

COPY ./resources/php.ini-${ENV_STATE} /usr/local/etc/php/php.ini

# Clean up
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www/html

# Copy existing application directory permissions
COPY --chown=www:www . /var/www/html

# Change current user to www
USER www

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]

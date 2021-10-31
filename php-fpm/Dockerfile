FROM php:8.0-fpm

COPY wait-for-it.sh /usr/bin/wait-for-it

RUN chmod +x /usr/bin/wait-for-it

RUN apt-get update && \
    apt-get install -y --no-install-recommends libssl-dev zlib1g-dev curl git unzip netcat libxml2-dev libpq-dev libzip-dev && \
    pecl install apcu && \
    docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql && \
    docker-php-ext-install -j$(nproc) zip opcache intl pdo_pgsql pgsql && \
    docker-php-ext-enable apcu pdo_pgsql sodium && \
    apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install memcached
RUN apt-get update && apt-get install -y libz-dev libmemcached-dev && rm -r /var/lib/apt/lists/*
RUN pecl install memcached
RUN echo extension=memcached.so >> /usr/local/etc/php/conf.d/memcached.ini

# Configure php dependency
RUN docker-php-ext-configure soap --enable-soap

# Install php dependencies
RUN docker-php-ext-install \
  pdo_mysql \
  pdo_pgsql \
  mysqli

RUN apt update && apt-get install libmagickwand-dev -y

# Install pecl
RUN pecl install xdebug
RUN pecl -v install redis && docker-php-ext-enable redis
RUN pecl -v install rar && docker-php-ext-enable rar
RUN pecl install imagick -o -f &&  docker-php-ext-enable imagick

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Install nodejs
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt update && apt install nodejs

# Install yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt update && apt install yarn

RUN apt update && apt-get install -y wget && rm -rf /var/lib/apt/lists/*
RUN wget https://get.symfony.com/cli/installer -O - | bash
RUN mv ~/.symfony/bin/symfony /usr/local/bin/symfony

RUN usermod -u 1000 www-data

# Add configs
ADD configs/20-xdebug.ini /usr/local/etc/php/conf.d/20-xdebug.ini
ADD configs/symfony.ini /usr/local/etc/php/conf.d/symfony.ini
ADD configs/symfony.pool.conf /usr/local/etc/php/conf.d/symfony.pool.conf

WORKDIR /var/www

EXPOSE 9000

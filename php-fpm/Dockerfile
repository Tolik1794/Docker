FROM php:8.1-fpm

RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list

# Install app from apt
RUN apt-get update && apt-get install -y  \
    nodejs  \
    yarn  \
    git  \
    unzip  \
    netcat  \
    wget  \
    nano  \
    ssh-client

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Install symfony
RUN wget https://get.symfony.com/cli/installer -O - | bash
RUN mv ~/.symfony/bin/symfony /usr/local/bin/symfony

# Add ssh
#RUN mkdir -p /root/.ssh
#ADD ssh/id_rsa /root/.ssh
#ADD ssh/id_rsa.pub /root/.ssh
#ADD ssh/known_hosts /root/.ssh
#RUN chmod -r /root/.ssh && chmod 600 /root/.ssh/id_rsa

# Install php extensions
RUN apt-get install -y --no-install-recommends \
    libssl-dev  \
    zlib1g-dev  \
    libxml2-dev  \
    libpq-dev  \
    libzip-dev  \
    libpng-dev  \
    libjpeg-dev  \
    libfreetype-dev  \
    libmagickwand-dev \
    libz-dev  \
    libsodium-dev  \
    libmemcached-dev

# Clear apt list and temp files
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install pecl dependency
RUN pecl install xdebug &&  \
    pecl install imagick -o -f &&  \
    pecl install apcu &&  \
    pecl install memcached-3.2.0

RUN pecl -v install redis && docker-php-ext-enable redis

# Configure php dependency
RUN docker-php-ext-configure soap &&  \
    docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql &&  \
    docker-php-ext-configure gd --with-freetype --with-jpeg

# Install php dependencies
RUN docker-php-ext-install -j$(nproc)  \
    pdo_mysql  \
    pdo_pgsql  \
    mysqli  \
    gd  \
    sodium  \
    pgsql  \
    intl  \
    opcache  \
    soap \
    zip

# Enable php dependencies
RUN docker-php-ext-enable imagick &&  \
    docker-php-ext-enable apcu &&  \
    docker-php-ext-enable memcached

RUN usermod -u 1000 www-data

# Add configs
ADD configs/20-xdebug.ini /usr/local/etc/php/conf.d/20-xdebug.ini
ADD configs/symfony.ini /usr/local/etc/php/conf.d/symfony.ini

WORKDIR /var/www

EXPOSE 9000

FROM php:8.4-fpm

WORKDIR /var/www/html

RUN sed -i \
    -e 's|http://deb.debian.org|https://deb.debian.org|g' \
    -e 's|http://security.debian.org|https://security.debian.org|g' \
    /etc/apt/sources.list.d/debian.sources \
    2>/dev/null || true

RUN apt-get update \
    -o Acquire::Retries=3 \
    && apt-get install -y --no-install-recommends \
    git \
    curl \
    unzip \
    zip \
    libzip-dev \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libonig-dev \
    libxml2-dev \
    libicu-dev \
    libpq-dev \
    && docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
    && docker-php-ext-install \
        pdo_mysql \
        pdo_pgsql \
        mbstring \
        exif \
        pcntl \
        bcmath \
        gd \
        intl \
        zip \
        opcache \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY . /var/www/html

RUN chown -R www-data:www-data /var/www/html

EXPOSE 9000

CMD ["php-fpm"]
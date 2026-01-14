############################
# NODE BUILDER (VITE ASSETS)
############################
FROM node:20-alpine AS node-builder

WORKDIR /app

# Copy only package files first (cache friendly)
COPY package.json package-lock.json ./

RUN npm ci

# Copy required frontend files
COPY resources ./resources
COPY vite.config.js ./

# Build assets
RUN npm run build


############################
# PHP APP IMAGE
############################
FROM php:8.3-fpm

# -------------------------
# SYSTEM DEPENDENCIES
# -------------------------
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    curl \
    zip \
    mariadb-client \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libzip-dev \
    libicu-dev \
    libonig-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install \
        pdo_mysql \
        mbstring \
        zip \
        exif \
        pcntl \
        bcmath \
        gd \
        intl \
        calendar \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# -------------------------
# COMPOSER
# -------------------------
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# -------------------------
# WORKDIR
# -------------------------
WORKDIR /var/www

# Copy full application
COPY . .

# Copy built frontend assets
COPY --from=node-builder /app/public/build ./public/build

# -------------------------
# PERMISSIONS
# -------------------------
RUN chown -R www-data:www-data storage bootstrap/cache public \
    && chmod -R 775 storage bootstrap/cache public

# -------------------------
# PHP DEPENDENCIES
# -------------------------
RUN composer install \
    --no-dev \
    --optimize-autoloader \
    --no-interaction

# -------------------------
# DEFAULT CMD
# -------------------------
CMD ["php-fpm"]

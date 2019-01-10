FROM alpine:latest AS base
RUN apk add --no-cache \
    nginx \
    php7-fpm \
    php7-json \
    php7-phar \
    php7-mbstring \
    php7-iconv \
    php7-openssl \
    php7-curl \
    php7-ctype \
    php7-dom \
    php7-pdo \
    php7-pdo_mysql \
    php7-xml \
    php7-simplexml \
    php7-tokenizer \
    php7-session

RUN adduser -D -g '' -G www-data www-data \
    && mkdir -p /var/www \
    && chown www-data:www-data /var/www

FROM base AS build
RUN apk add --no-cache \
    npm \
    curl

USER www-data
WORKDIR /var/www

########################################################################################################################

# Install composer with prestissimo plugin
RUN curl -sS https://getcomposer.org/installer | php \
    && php ./composer.phar global require hirak/prestissimo

########################################################################################################################

# Install node.js dependencies
COPY package* ./
RUN npm install

# Install composer dependencies
COPY composer* ./
RUN php ./composer.phar install \
        --no-dev \
        --no-autoloader \
        --ignore-platform-reqs \
        --no-scripts

########################################################################################################################

# Build assets
COPY --chown=www-data:www-data assets /var/www/assets
COPY --chown=www-data:www-data public /var/www/public
COPY --chown=www-data:www-data webpack.config.js /var/www/webpack.config.js
RUN npm run-script build \
    && rm -rf assets \
    && rm -rf node_modules

# Copy entire application, generate composer autoload and warmup the cache.
COPY --chown=www-data:www-data bin /var/www/bin
COPY --chown=www-data:www-data config /var/www/config
COPY --chown=www-data:www-data data /var/www/data
COPY --chown=www-data:www-data src /var/www/src
COPY --chown=www-data:www-data templates /var/www/templates
COPY --chown=www-data:www-data translations /var/www/translations
COPY --chown=www-data:www-data .env /var/www/.env
RUN echo "APP_ENV=prod" > .env.local
RUN php ./composer.phar dumpautoload \
        --no-dev \
        --optimize \
    && php ./composer.phar run \
        post-install-cmd

########################################################################################################################

FROM base AS run
COPY --from=build --chown=www-data:www-data /var/www /var/www
RUN mkdir -p /var/www/var/cache/prod && chown -R www-data:www-data /var/www/var/

USER root
RUN rm /etc/nginx/conf.d/default.conf
COPY docker/nginx/nginx.conf    /etc/nginx/nginx.conf
COPY docker/nginx/symfony.conf  /etc/nginx/conf.d/symfony.conf

COPY docker/php-fpm/symfony.ini /etc/php7/conf.d/
COPY docker/php-fpm/symfony.ini /etc/php7/cli/conf.d/
COPY docker/php-fpm/xdebug.ini  /etc/php7/conf.d/
COPY docker/php-fpm/symfony.pool.conf /etc/php7/php-fpm.d/

COPY docker/entry-point.sh entry-point.sh
CMD ["/bin/sh", "./entry-point.sh"]

EXPOSE 80

ARG PHP_VERSION=$PHP_VERSION

FROM php:${PHP_VERSION}-fpm-alpine AS init

ARG APP_ENV=$APP_ENV

ENV PHP_VERSION $PHP_VERSION
ENV APP_ENV $APP_ENV

# persistent / runtime deps
RUN apk add --no-cache \
		acl \
		fcgi \
		file \
		gettext \
		git \
        bash \
	;

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		icu-dev \
		libzip-dev \
		zlib-dev \
	; \
	\
	docker-php-ext-configure zip; \
	docker-php-ext-install -j$(nproc) \
		intl \
		zip \
	; \
	pecl install \
		apcu \
	; \
	pecl clear-cache; \
	docker-php-ext-enable \
		apcu \
		opcache \
	; \
	\
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-cache --virtual .api-phpexts-rundeps $runDeps; \
	\
	apk del .build-deps

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN ln -s $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini
COPY docker/php/conf.d/init.${APP_ENV}.ini $PHP_INI_DIR/conf.d/init.ini

WORKDIR /srv/api

COPY src src/

COPY docker/php/docker-entrypoint.sh /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint

ENTRYPOINT ["docker-entrypoint"]
CMD ["php-fpm"]

FROM init AS init_dev
RUN set -eux; \
	apk add --no-cache --virtual .build-deps linux-headers $PHPIZE_DEPS; \
	pecl install xdebug; \
	docker-php-ext-enable xdebug; \
	apk del .build-deps
EXPOSE 9003

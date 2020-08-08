ARG PHP_VERSION=7.4.6
ARG APCU_VERSION=5.1.16

# build for production
ARG APP_ENV=prod

FROM php:${PHP_VERSION}-fpm-buster

RUN useradd -m -o -u 1000 docker;

ENV TZ=Europe/Madrid
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN set -eux;
RUN mkdir -p /usr/share/man/man1

RUN apt update \
	&& apt install -y --fix-missing --no-install-recommends \
		$PHPIZE_DEPS \
		software-properties-common \
		libicu-dev \
		libzip-dev  \
		zlib1g-dev  \
		apt-utils \
		pdftk \
		git \
		libssl-dev \
		wget \
		gnupg2 \	
		unzip \
		libmagickwand-dev \
		rclone \
		cron \
	&& docker-php-ext-install -j$(nproc) \
       		intl \
       		mysqli \
       		pdo \
       		pdo_mysql \
       		zip \
       		bcmath \
	&& apt clean \
	;

RUN wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.buster_amd64.deb \
	&& apt install -y ./wkhtmltox_0.12.5-1.buster_amd64.deb \
	&& rm ./wkhtmltox_0.12.5-1.buster_amd64.deb

RUN pecl install \
#	apcu-5.1.16 \
	imagick \
	raphf \
	propro \
	xdebug \
	&& printf "\n" | \
	docker-php-ext-enable imagick \
	raphf \
	propro \
	xdebug \
	&& pecl clear-cache \
    ;
    


# Install Blackfire CLI tool and PHP Probe
RUN wget -q -O - https://packages.blackfire.io/gpg.key | apt-key add -
RUN echo "deb http://packages.blackfire.io/debian any main" | tee /etc/apt/sources.list.d/blackfire.list
RUN apt-get update && apt-get install -y blackfire-agent blackfire-php


RUN mkdir -p /usr/local/etc/php/conf.d/disabled
RUN rm /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
COPY docker/conf.d/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/disabled/docker-php-ext-xdebug.ini


COPY docker/conf.d/api-platform.ini $PHP_INI_DIR/conf.d/api-platform.ini
RUN ln -s $PHP_INI_DIR/php.ini-production $PHP_INI_DIR/php.ini

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
# https://getcomposer.org/doc/03-cli.md#composer-allow-superuser
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV PATH="${PATH}:/root/.composer/vendor/bin"

RUN set -eux; \
	composer global require "hirak/prestissimo:^0.3" --prefer-dist --no-progress --no-suggest --classmap-authoritative; \
	composer clear-cache


WORKDIR /srv/app
#USER www-data

#VOLUME /srv/app/var
COPY docker/docker-entrypoint.sh /usr/local/bin/docker-entrypoint
RUN chmod +x /usr/local/bin/docker-entrypoint

ENTRYPOINT ["docker-entrypoint"]
CMD ["php-fpm"]

# from https://www.drupal.org/docs/8/system-requirements/drupal-8-php-requirements
FROM php:7.4-apache-buster
# TODO switch to buster once https://github.com/docker-library/php/issues/865 is resolved in a clean way (either in the PHP image or in PHP itself)

# install the PHP extensions we need
RUN set -eux; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libzip-dev \
	; \
	\
	docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg=/usr \
	; \
	docker-php-ext-configure exif; \
	docker-php-ext-install -j "$(nproc)" \
		exif \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
	\
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN apt-get update && \
	apt-get -y install git && \
	git clone https://github.com/Jan-E/uploadprogress.git /tmp/php-uploadprogress && \
	cd /tmp/php-uploadprogress && \
	phpize && \
	./configure --prefix=/usr && \
	make && \
	make install && \
	echo 'extension=uploadprogress.so' > "${PHP_INI_DIR}/conf.d/uploadprogress.ini" && \
	echo 'allow_url_include=1' > "${PHP_INI_DIR}/conf.d/mmed.ini" && \
	echo 'upload_max_filesize=50M' >> "${PHP_INI_DIR}/conf.d/mmed.ini" && \
	echo 'post_max_size=100M' >> "${PHP_INI_DIR}/conf.d/mmed.ini" && \
	cd /tmp && \
	git clone https://github.com/longxinH/xhprof.git && \
	cd /tmp/xhprof/extension && \
	phpize && \
	./configure && make && make install && \
	echo '[xhprof]' > "${PHP_INI_DIR}/conf.d/xhprof.ini" && \
	echo 'extension = xhprof.so' >> "${PHP_INI_DIR}/conf.d/xhprof.ini" && \
	echo 'xhprof.output_dir = /xhprof' >> "${PHP_INI_DIR}/conf.d/xhprof.ini" && \
	mkdir /xhprof && chown www-data:www-data /xhprof && \
	curl -so /tmp/composer-installer.sig https://composer.github.io/installer.sig && \
	curl -so /tmp/composer-setup.php https://getcomposer.org/installer && \
	php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') !== trim(file_get_contents('/tmp/composer-installer.sig'))) {unlink('/tmp/composer-setup.php'); echo 'Invalid installer' . PHP_EOL; exit(1);}" && \
	php /tmp/composer-setup.php --no-ansi --filename=composer --snapshot && \
	mv composer /usr/local/bin && \
	rm -rf /tmp/* /var/lib/apt/lists/*

# https://www.drupal.org/node/3060/release
ENV DRUPAL_VERSION 7.72
ENV DRUPAL_MD5 ed967195ce0e78bf2ab7245aaf0649d6

WORKDIR /var/www/html

# vim:set ft=dockerfile:

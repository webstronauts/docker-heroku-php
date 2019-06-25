FROM heroku/heroku:18

# Which versions?
ARG HEROKU_CEDAR_VERSION=18
ARG PHP_VERSION

# Internally, we arbitrarily use port 3000
ENV PORT 3000

ENV HEROKU_CEDAR_VERSION=$HEROKU_CEDAR_VERSION
ENV PHP_VERSION=$PHP_VERSION
ENV APCU_EXT_VERSION=5.1.17
ENV REDIS_EXT_VERSION=4.3.0
ENV IMAGICK_EXT_VERSION=3.4.3
ENV HTTPD_VERSION=2.4.39
ENV NGINX_VERSION=1.14.2
ENV NODE_VERSION=10.15.0
ENV COMPOSER_VERSION=1.8.5
ENV YARN_VERSION=1.13.0

ENV COMPOSER_HOME /app/.composer/

# make apt non-interactive
RUN echo 'APT::Get::Assume-Yes "true";' > /etc/apt/apt.conf.d/90docker \
 && echo 'DPkg::Options "--force-confnew";' >> /etc/apt/apt.conf.d/90docker

ENV DEBIAN_FRONTEND=noninteractive

# Create some needed directories
RUN mkdir -p /app/.heroku/php /app/.heroku/node /app/.profile.d $COMPOSER_HOME
WORKDIR /app/user

# Locate our binaries
ENV PATH /app/.heroku/php/bin:/app/.heroku/php/sbin:/app/.heroku/node/bin/:/app/user/node_modules/.bin:/app/user/vendor/bin:$PATH

# Install and configure Apache
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-${HEROKU_CEDAR_VERSION}-stable/apache-${HTTPD_VERSION}.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://raw.githubusercontent.com/heroku/heroku-buildpack-php/5a770b914549cf2a897cbbaf379eb5adf410d464/conf/apache2/httpd.conf.default > /app/.heroku/php/etc/apache2/httpd.conf \
 # FPM socket permissions workaround when run as root
 && echo "\n\
Group root\n\
" >> /app/.heroku/php/etc/apache2/httpd.conf

# Install and configure Nginx
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-${HEROKU_CEDAR_VERSION}-stable/nginx-${NGINX_VERSION}.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://raw.githubusercontent.com/heroku/heroku-buildpack-php/5a770b914549cf2a897cbbaf379eb5adf410d464/conf/nginx/nginx.conf.default > /app/.heroku/php/etc/nginx/nginx.conf \
 # FPM socket permissions workaround when run as root
 && echo "\n\
user nobody root;\n\
" >> /app/.heroku/php/etc/nginx/nginx.conf

# Install and configure PHP
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-$HEROKU_CEDAR_VERSION-stable/php-$PHP_VERSION.tar.gz | tar xz -C /app/.heroku/php \
 && mkdir -p /app/.heroku/php/etc/php/conf.d \
 && ZEND_MODULE_API_VERSION=$(php -i | grep 'PHP API' | sed -e 's/PHP API => //') \
 && curl --silent --location https://raw.githubusercontent.com/heroku/heroku-buildpack-php/5a770b914549cf2a897cbbaf379eb5adf410d464/conf/php/php.ini > /app/.heroku/php/etc/php/php.ini \
 && curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-${HEROKU_CEDAR_VERSION}-stable/extensions/no-debug-non-zts-${ZEND_MODULE_API_VERSION}/apcu-${APCU_EXT_VERSION}.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-${HEROKU_CEDAR_VERSION}-stable/extensions/no-debug-non-zts-${ZEND_MODULE_API_VERSION}/redis-${REDIS_EXT_VERSION}.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-${HEROKU_CEDAR_VERSION}-stable/extensions/no-debug-non-zts-${ZEND_MODULE_API_VERSION}/imagick-${IMAGICK_EXT_VERSION}.tar.gz | tar xz -C /app/.heroku/php \
 # Enable all optional exts
 && echo "\n\
memory_limit = 512M \n\
user_ini.cache_ttl = 30 \n\
opcache.enable = 0 \n\
extension=apcu.so \n\
extension=bcmath.so \n\
extension=calendar.so \n\
extension=exif.so \n\
extension=ftp.so \n\
extension=gd.so\n\
extension=gettext.so \n\
extension=gmp.so \n\
extension=imagick.so \n\
extension=intl.so \n\
extension=mbstring.so \n\
extension=pcntl.so \n\
extension=redis.so \n\
extension=shmop.so \n\
extension=soap.so \n\
extension=sqlite3.so \n\
extension=xmlrpc.so \n\
extension=xsl.so\n\
" >> /app/.heroku/php/etc/php/php.ini

# Install xdebug (but don't enable)
RUN apt-get update \
 && apt-get -y install gcc make autoconf libc-dev pkg-config \
 && pecl install xdebug \
 && rm -rf /var/lib/apt/lists/*

# Install Composer
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/composer-$COMPOSER_VERSION.tar.gz | tar xz -C /app/.heroku/php

# Install Node
RUN curl --silent --location https://s3pository.heroku.com/node/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz | tar --strip-components=1 -xz -C /app/.heroku/node

# Install Yarn
RUN curl --silent --location https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz | tar --strip-components=1 -xz -C /app/.heroku/node

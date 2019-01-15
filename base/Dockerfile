FROM heroku/heroku:18

# Internally, we arbitrarily use port 3000
ENV PORT 3000

# Which versions?
ENV PHP_VERSION 7.2.14
ENV APCU_EXT_VERSION 5.1.16
ENV REDIS_EXT_VERSION 4.2.0
ENV IMAGICK_EXT_VERSION 3.4.3
ENV HTTPD_VERSION 2.4.37
ENV NGINX_VERSION 1.8.1
ENV NODE_VERSION 10.15.0
ENV COMPOSER_VERSION 1.8.0
ENV YARN_VERSION 1.13.0

# Create some needed directories
RUN mkdir -p /app/.heroku/php /app/.heroku/node /app/.profile.d
WORKDIR /app/user

# Locate our binaries
ENV PATH /app/.heroku/php/bin:/app/.heroku/php/sbin:/app/.heroku/node/bin/:/app/user/node_modules/.bin:/app/user/vendor/bin:$PATH

# Install and configure Apache
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/apache-$HTTPD_VERSION.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://raw.githubusercontent.com/heroku/heroku-buildpack-php/5a770b914549cf2a897cbbaf379eb5adf410d464/conf/apache2/httpd.conf.default > /app/.heroku/php/etc/apache2/httpd.conf \
 # FPM socket permissions workaround when run as root
 && echo "\n\
Group root\n\
" >> /app/.heroku/php/etc/apache2/httpd.conf

# Install and configure Nginx
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/nginx-$NGINX_VERSION.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://raw.githubusercontent.com/heroku/heroku-buildpack-php/5a770b914549cf2a897cbbaf379eb5adf410d464/conf/nginx/nginx.conf.default > /app/.heroku/php/etc/nginx/nginx.conf \
 # FPM socket permissions workaround when run as root
 && echo "\n\
user nobody root;\n\
" >> /app/.heroku/php/etc/nginx/nginx.conf

# Install and configure PHP
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/php-$PHP_VERSION.tar.gz | tar xz -C /app/.heroku/php \
 && mkdir -p /app/.heroku/php/etc/php/conf.d \
 && curl --silent --location https://raw.githubusercontent.com/heroku/heroku-buildpack-php/5a770b914549cf2a897cbbaf379eb5adf410d464/conf/php/php.ini > /app/.heroku/php/etc/php/php.ini \
 && curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/extensions/no-debug-non-zts-20170718/apcu-$APCU_EXT_VERSION.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/extensions/no-debug-non-zts-20170718/redis-$REDIS_EXT_VERSION.tar.gz | tar xz -C /app/.heroku/php \
 && curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/extensions/no-debug-non-zts-20170718/imagick-$IMAGICK_EXT_VERSION.tar.gz | tar xz -C /app/.heroku/php \
 # Enable all optional exts
 && echo "\n\
user_ini.cache_ttl = 30 \n\
opcache.enable = 0 \n\
extension=apcu.so \n\
extension=bcmath.so \n\
extension=calendar.so \n\
extension=exif.so \n\
extension=ftp.so \n\
extension=gd.so\n\
extension=gettext.so \n\
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
RUN apt-get update && apt-get -y install gcc make autoconf libc-dev pkg-config \
 && pecl install xdebug

# Install Composer
RUN curl --silent --location https://lang-php.s3.amazonaws.com/dist-heroku-18-stable/composer-$COMPOSER_VERSION.tar.gz | tar xz -C /app/.heroku/php

# Install Node
RUN curl --silent --location https://s3pository.heroku.com/node/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz | tar --strip-components=1 -xz -C /app/.heroku/node

# Install Yarn
RUN curl --silent --location https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz | tar --strip-components=1 -xz -C /app/.heroku/node

# Install Chrome WebDriver
RUN CHROMEDRIVER_VERSION=`curl -sS chromedriver.storage.googleapis.com/LATEST_RELEASE` \
 && mkdir -p /opt/chromedriver-$CHROMEDRIVER_VERSION \
 && curl -sS -o /tmp/chromedriver_linux64.zip http://chromedriver.storage.googleapis.com/$CHROMEDRIVER_VERSION/chromedriver_linux64.zip \
 && unzip -qq /tmp/chromedriver_linux64.zip -d /opt/chromedriver-$CHROMEDRIVER_VERSION \
 && rm /tmp/chromedriver_linux64.zip \
 && chmod +x /opt/chromedriver-$CHROMEDRIVER_VERSION/chromedriver \
 && ln -fs /opt/chromedriver-$CHROMEDRIVER_VERSION/chromedriver /usr/local/bin/chromedriver

# Install Google Chrome
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
 && echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list \
 && apt-get update -qqy \
 && apt-get -qqy install google-chrome-stable \
 && rm /etc/apt/sources.list.d/google-chrome.list \
 && rm -rf /var/lib/apt/lists/*

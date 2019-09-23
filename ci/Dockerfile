ARG PHP_ALIAS
FROM webstronauts/heroku-php:${PHP_ALIAS}

# Which versions?
ENV DOCKERIZE_VERSION v0.6.1

# Add dockerize utilities
RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
 && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
 && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

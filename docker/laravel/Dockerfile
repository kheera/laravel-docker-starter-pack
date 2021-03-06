FROM ubuntu:latest

MAINTAINER Shane Nelson

# http://knowledgebasement.com/how-to-run-sites-securely-with-apache-and-php-fpm-on-ubuntu-16-04-lts/
# May have to remove mod_php_apache to get this working
# Set proper owner in:  /etc/php/7.2/fpm/pool.d/www.conf

# Stops apt-get from asking questions, like "what timezone do you live in?"
ARG PROJECT_NAME
ENV DEBIAN_FRONTEND noninteractive
ENV PROJECT_NAME=${PROJECT_NAME}

# Docker recommends all apt-get commands occur in one RUN command
# and they should be listed alphabetically so as to prevent installing the same package twice
RUN apt-get update
RUN apt-get install -y \
    apache2 \
    curl \
    g++ \
    gcc \
    gnupg \
    make \
    vim \
    php \
    php-common \
    php-curl \
    php-mbstring \
    php-mysql \
    php-fpm \
    php-xml \
    php-zip \
    rsync \
    unzip

RUN a2enmod rewrite
RUN a2enmod proxy_fcgi setenvif
RUN a2enconf php7.2-fpm
RUN a2enmod ssl

RUN mkdir /etc/self-signed/
RUN printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth" > /etc/self-signed/openssl.conf
RUN openssl req -x509 -out /etc/self-signed/localhost.crt -keyout /etc/self-signed/localhost.key \
  -newkey rsa:2048 -nodes -sha256 \
  -subj '/CN=localhost' -extensions EXT \
  -config /etc/self-signed/openssl.conf

# Install node (so we can use npm)
WORKDIR /usr/local/sbin
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash; \
    apt-get install -y nodejs

RUN curl -sS https://getcomposer.org/installer \
    | php && mv composer.phar /usr/local/bin/composer \
    && composer global require hirak/prestissimo --no-plugins --no-scripts

# Modify apache config for Laravel
COPY ./docker/laravel/apache/000-default.conf /etc/apache2/sites-enabled/
COPY ./docker/laravel/apache/default-ssl.conf /etc/apache2/sites-enabled/

# Redirects log files to docker console
RUN ln -sf /proc/self/fd/1 /var/log/apache2/access.log && \
    ln -sf /proc/self/fd/1 /var/log/apache2/error.log

WORKDIR /var/www
COPY ./docker/laravel/start.sh /usr/local/sbin/start.sh

WORKDIR /var/www/html

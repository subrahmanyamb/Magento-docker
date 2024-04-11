# Use the PHP 8.1 FPM image as the base image
FROM php:8.1-fpm

# Set the timezone (change 'TIMEZONE' to your desired timezone)
ARG timezone
RUN ln -snf /usr/share/zoneinfo/$TIMEZONE /etc/localtime && echo $TIMEZONE > /etc/timezone

# Update and upgrade system packages
RUN apt-get update && apt-get upgrade -y

# Install system dependencies
RUN apt-get update && apt-get install -y libxml2-dev libgd-dev libicu-dev zlib1g-dev libssl-dev pkg-config libpq-dev libzip-dev wget gnupg git unzip libxslt-dev

# Install PHP extensions
RUN set -xe \
    && docker-php-ext-configure intl \
    && docker-php-ext-install \
        intl \
        opcache \
        zip \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        bcmath \
        xsl \
    && pecl install \
        apcu \
        mongodb \
        redis \
    && docker-php-ext-enable \
        apcu \
        mongodb \
        redis

# Install and enable GD extension
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install gd

# Install SOAP extension
RUN docker-php-ext-install soap

# Install Sockets extension
RUN docker-php-ext-install sockets
# Continue with other extensions

COPY php.ini /usr/local/etc/php/php.ini


COPY --from=composer:2.2 /usr/bin/composer /usr/bin/composer
# https://getcomposer.org/doc/03-cli.md#composer-allow-superuser
ENV COMPOSER_ALLOW_SUPERUSER 1
# create composer cache directory

RUN mkdir -p /var/www/.composer && chown -R www-data /var/www/.composer


# npm & node
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash
#RUN apt-get install -y nodejs \
#  && update-alternatives --install /usr/bin/node node /usr/bin/nodejs 10
RUN apt install ssh -y
# build tools
RUN apt-get install -y build-essential
#RUN apt-get install -y ssh

# bugfix: remove cmdtest to install yarn correctly.
RUN apt-get remove -y cmdtest

# yarn package manager
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
  && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install -y yarn

RUN curl -sS https://get.symfony.com/cli/installer | bash
RUN mv /root/.symfony5/bin/symfony /usr/local/bin/symfony

RUN curl -Lsf 'https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz' | tar -C '/usr/local' -xvzf -
ENV PATH /usr/local/go/bin:$PATH
RUN go get github.com/mailhog/mhsendmail
RUN cp /root/go/bin/mhsendmail /usr/bin/mhsendmail
RUN echo 'sendmail_path = /usr/bin/mhsendmail --smtp-addr mailhog:1025' > /usr/local/etc/php/php.ini

WORKDIR /srv
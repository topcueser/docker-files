FROM php:7.0-apache

# install pecl
RUN curl -O http://pear.php.net/go-pear.phar \
    ; /usr/local/bin/php -d detect_unicode=0 go-pear.phar

# PHP extensions
RUN apt-get update && apt-get install -y \
        unzip \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libaio1 \
        vim \
        libzip-dev \
        libpq-dev \
        libxml2-dev \
        git \
        libcurl4-openssl-dev \
        pkg-config \
        libssl-dev \
    && docker-php-ext-install -j$(nproc) iconv gettext mcrypt dom zip \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install pdo pdo_mysql \
    && docker-php-ext-install pdo_pgsql \
    && docker-php-ext-install pgsql

#Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

# apache configurations, mod rewrite
RUN ln -s /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled/rewrite.load
RUN a2enmod rewrite

RUN cd /tmp && curl -L https://raw.githubusercontent.com/topcueser/docker-files/master/oci8-instantclient/instantclient-basiclite-linux.x64-19.3.0.0.0dbru.zip -O
RUN cd /tmp && curl -L https://raw.githubusercontent.com/topcueser/docker-files/master/oci8-instantclient/instantclient-sdk-linux.x64-19.3.0.0.0dbru.zip -O
RUN cd /tmp && curl -L https://raw.githubusercontent.com/topcueser/docker-files/master/oci8-instantclient/instantclient-sqlplus-linux.x64-19.3.0.0.0dbru.zip -O

RUN unzip /tmp/instantclient-basiclite-linux.x64-19.3.0.0.0dbru.zip -d /usr/local/
RUN unzip /tmp/instantclient-sdk-linux.x64-19.3.0.0.0dbru.zip -d /usr/local/
RUN unzip /tmp/instantclient-sqlplus-linux.x64-19.3.0.0.0dbru.zip -d /usr/local/

RUN ln -s /usr/local/instantclient_19_3 /usr/local/instantclient
RUN ln -s /usr/local/instantclient/lib* /usr/lib
RUN ln -s /usr/local/instantclient/sqlplus /usr/bin/sqlplus

RUN echo 'export ORACLE_HOME="/usr/local/instantclient"' >> /root/.bashrc
RUN echo 'export LD_LIBRARY_PATH="/usr/local/instantclient"' >> /root/.bashrc
RUN echo 'umask 002' >> /root/.bashrc

RUN pecl channel-update pecl.php.net
RUN echo 'instantclient,/usr/local/instantclient' | pecl install oci8-2.2.0
RUN echo "extension=oci8.so" > /usr/local/etc/php/conf.d/php-oci8.ini

RUN echo 'install mongodb' | pecl install mongodb
RUN echo "extension=mongodb.so" > /usr/local/etc/php/conf.d/mongodb.ini

RUN mkdir -p /var/www
WORKDIR /var/www
RUN chown www-data:www-data /var/www/

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
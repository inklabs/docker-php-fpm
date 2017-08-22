FROM php:5.6-fpm

LABEL maintainer="Jamie Isaacs <pdt256@gmail.com>"

RUN apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y --force-yes \
        libcurl4-openssl-dev \
        libfreetype6-dev \
        libjpeg-dev \
        libjpeg62-turbo-dev \
        libpng12-dev \
        libpq-dev \
        libgd3 \
        libperl4-corelibs-perl \
        libvpx1 \
        libxpm4 \
        libicu-dev \
        libmcrypt-dev \
        libbz2-dev \
        libxml2-dev \
        libxpm-dev \
        libxrender1 \
        libxext6 \
        libtool \
        libgearman-dev \
        unzip \
        automake \
        gcc \
        && apt-get clean

# Install php extensions
RUN docker-php-ext-install \
        bcmath \
        bz2 \
        calendar \
        curl \
        exif \
        gettext \
        hash \
        mbstring \
        mcrypt \
        mysql \
        opcache \
        pdo_mysql \
        sockets \
        zip \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/lib/x86_64-linux-gnu/ --with-jpeg-dir=/usr/lib/x86_64-linux-gnu/ --with-xpm-dir=/usr/include && docker-php-ext-install gd \
    && pecl install igbinary && docker-php-ext-enable igbinary

# Install Redis
ENV REDIS_NAME=redis-2.2.5
RUN curl -fsSL http://pecl.php.net/get/$REDIS_NAME.tgz -o $REDIS_NAME.tgz \
    && tar zxf $REDIS_NAME.tgz \
    && ( \
        cd $REDIS_NAME \
        && phpize \
        && ./configure --enable-redis-igbinary \
        && make && make install \
    ) \
    && rm -r $REDIS_NAME \
    && docker-php-ext-enable redis

# Install lib-maxmind-db
ENV MAXMIND_VER=0.5.3
RUN curl -fsSL "https://github.com/maxmind/libmaxminddb/archive/$MAXMIND_VER.zip" -o libmaxminddb-$MAXMIND_VER.zip \
    && unzip libmaxminddb-$MAXMIND_VER.zip \
    && ( \
        cd libmaxminddb-$MAXMIND_VER \
        && ./bootstrap \
        && ./configure \
        && make && make install \
    ) \
    && rm -r libmaxminddb-$MAXMIND_VER \
    && ldconfig /usr/local/lib/

# Install maxmind-reader
ENV MAXMINDREADER_VER=0.3.2
RUN curl -fsSL "https://github.com/maxmind/MaxMind-DB-Reader-php/archive/v$MAXMINDREADER_VER.zip" -o MaxMind-DB-Reader-php-$MAXMINDREADER_VER.zip \
    && unzip MaxMind-DB-Reader-php-$MAXMINDREADER_VER.zip \
    && ( \
        cd MaxMind-DB-Reader-php-$MAXMINDREADER_VER/ext \
        && phpize \
        && ./configure \
        && make && make install \
    ) \
    && rm -r MaxMind-DB-Reader-php-$MAXMINDREADER_VER \
    && docker-php-ext-enable maxminddb

# Install php-gearman
ENV PHPGEARMAN_NAME=gearman-1.1.2
RUN curl -fsSL "http://pecl.php.net/get/$PHPGEARMAN_NAME.tgz" -o $PHPGEARMAN_NAME.tgz \
    && tar zxf $PHPGEARMAN_NAME.tgz \
    && ( \
        cd $PHPGEARMAN_NAME \
        && phpize \
        && ./configure \
        && make && make install \
    ) \
    && rm -r $PHPGEARMAN_NAME \
    && docker-php-ext-enable gearman

RUN touch /var/log/php_errors.log \
    && chown www-data.www-data /var/log/php_errors.log \
    && mkdir -p /var/log/php-fpm \
    && chown www-data.www-data /var/log/php-fpm

COPY ./etc/php-conf.d/* /usr/local/etc/php/conf.d/
COPY ./etc/php-fpm.conf /usr/local/etc/php-fpm.conf
COPY ./etc/php-fpm.d/ /usr/local/etc/php-fpm.d

EXPOSE 9000
CMD ["php-fpm"]
USER www-data

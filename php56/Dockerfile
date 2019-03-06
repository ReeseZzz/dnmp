FROM php:5.6-fpm

# Version
ENV PHPREDIS_VERSION 4.0.0
ENV HIREDIS_VERSION 0.13.3
ENV SWOOLE_VERSION 2.0.3

# Timezone
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' > /etc/timezone

COPY ./sources.list /etc/apt/sources.list

# Libs
RUN apt-get update \
    && apt-get install -y \
        curl \
        wget \
        zip \
        libz-dev \
        libssl-dev \
        libnghttp2-dev \
        libpcre3-dev \
    && apt-get clean \
    && apt-get autoremove

# Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://packagist.phpcomposer.com \
    && composer self-update --clean-backups

# PDO extension
RUN docker-php-ext-install zip mysqli pdo_mysql

# Redis extension
RUN wget http://pecl.php.net/get/redis-${PHPREDIS_VERSION}.tgz -O /tmp/redis.tar.tgz \
    && pecl install /tmp/redis.tar.tgz \
    && rm -rf /tmp/redis.tar.tgz \
    && docker-php-ext-enable redis

#安装 Composer
ENV COMPOSER_HOME /root/composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV PATH $COMPOSER_HOME/vendor/bin:$PATH

# Swoole extension
RUN wget https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz -O swoole.tar.gz \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && rm swoole.tar.gz \
    && ( \
        cd swoole \
        && phpize \
        && ./configure \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r swoole \
    && docker-php-ext-enable swoole

WORKDIR /www

CMD ["php-fpm"]

EXPOSE 9000
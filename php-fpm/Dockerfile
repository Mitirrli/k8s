# ------------------------------------------------------------------------------------
#                                  Local Dockerfile
# ------------------------------------------------------------------------------------
# @build-example docker-compose up -d
# ------------------------------------------------------------------------------------
FROM php:7.4.8-fpm-alpine

# 使用国内镜像
RUN echo http://mirrors.aliyun.com/alpine/v3.12/main>/etc/apk/repositories \
    && echo  http://mirrors.aliyun.com/alpine/v3.12/community>>/etc/apk/repositories

# 安装必须的包
RUN apk add --no-cache oniguruma-dev \
    curl-dev \
    libxml2-dev \
    libzip-dev \
    libpng-dev freetype \
    libpng \
    openssl-dev \
    libffi-dev \
    libjpeg-turbo \
    freetype-dev \
    libpng-dev \
    jpeg-dev \
    libjpeg \
    libjpeg-turbo-dev \
    gcc \
    g++ \
    make \
    autoconf \
    icu-dev

# 增加GD库
RUN docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/

# 安装一些必备的扩展
RUN docker-php-ext-install gd bcmath pdo_mysql mysqli opcache pcntl mbstring sockets

# phpredis及swoole版本
ENV PHPREDIS_VERSION=5.2.2 \
    XLSWRITER_VERSION=1.3.6

# 安装composer
RUN wget https://mirrors.aliyun.com/composer/composer.phar -O /usr/local/bin/composer \
    && chmod a+x /usr/local/bin/composer \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer \
    && composer self-update --clean-backups \
# 安装redis扩展
    && wget http://pecl.php.net/get/redis-${PHPREDIS_VERSION}.tgz -O /tmp/redis.tar.tgz \
    && pecl install /tmp/redis.tar.tgz \
    && rm -rf /tmp/redis.tar.tgz \
    && docker-php-ext-enable redis \
# 安装xlswriter扩展
    && curl -fsSL "https://pecl.php.net/get/xlswriter-${XLSWRITER_VERSION}.tgz" -o xlswriter.tgz \
    && mkdir -p /tmp/xlswriter \
    && tar -xf xlswriter.tgz -C /tmp/xlswriter --strip-components=1 \
    && rm xlswriter.tgz \
    && cd /tmp/xlswriter \
    && phpize \
    && ./configure --enable-reader \
    && make -j$(nproc) \
    && make install \
    && rm -rf /xlswriter \
    && docker-php-ext-enable xlswriter

# Enable Opcache
RUN { \
        echo 'opcache.memory_consumption=256'; \
        echo 'opcache.interned_strings_buffer=64'; \
        echo 'opcache.max_accelerated_files=30000'; \
        echo 'opcache.validate_timestamps=0'; \
        echo 'opcache.fast_shutdown=1'; \
    } > /usr/local/etc/php/conf.d/opcache.ini

# Upload.ini
COPY upload.ini /usr/local/etc/php/conf.d

# 创建slow目录
RUN mkdir /usr/local/log/

# 复制配置文件到 /usr/local
COPY etc /usr/local/etc

# 设置工作目录
WORKDIR /var/www/html

# RUNTIME权限
RUN chown -R www-data:www-data runtime

ENTRYPOINT ["php-fpm", "-R"]
FROM tetraweb/php:7.0

MAINTAINER Joshua Gigg <giggsey@gmail.com>

ENV XDEBUG_VERSION 2.4.1

# Install php extensions
RUN buildDeps=" \
        libmemcached-dev \
        librabbitmq-dev \
        zlib1g-dev \
    " \
    && phpModules=" \
        xdebug \
    " \
    && echo "deb http://httpredir.debian.org/debian jessie contrib non-free" > /etc/apt/sources.list.d/additional.list \
    && apt-get install -y $buildDeps --no-install-recommends \
    && docker-php-source extract \
    && cd /usr/src/php/ext/ \
    && curl -L http://xdebug.org/files/xdebug-$XDEBUG_VERSION.tgz | tar -zxf - \
    && mv xdebug-$XDEBUG_VERSION xdebug \
    && docker-php-ext-install $phpModules \
    && printf "\n" | pecl install amqp \
    && for ext in $phpModules; do \
           rm -f /usr/local/etc/php/conf.d/docker-php-ext-$ext.ini; \
       done \
    && git clone https://github.com/websupport-sk/pecl-memcache.git /tmp/memcache/ \
    && cd /tmp/memcache/ \
    && git reset --hard df7735e \
    && phpize \
    && ./configure && make && make install \
    && rm -rf /tmp/memcache/ \
    && docker-php-source delete \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["php", "-a"]

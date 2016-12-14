FROM tetraweb/php:7.0

MAINTAINER Joshua Gigg <giggsey@gmail.com>

ENV MEMCACHE_REVISION 'df7735e'

# Install php extensions
RUN buildDeps=" \
        libmemcached-dev \
        zlib1g-dev \
    " \
    && echo "deb http://httpredir.debian.org/debian jessie contrib non-free" > /etc/apt/sources.list.d/additional.list \
    && apt-get install -y $buildDeps --no-install-recommends \
    && docker-php-source extract \
    && git clone https://github.com/websupport-sk/pecl-memcache.git /tmp/memcache/ \
    && cd /tmp/memcache/ \
    && git reset --hard $MEMCACHE_REVISION \
    && phpize \
    && ./configure && make && make install \
    && rm -rf /tmp/memcache/ \
    && docker-php-source delete \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["php", "-a"]

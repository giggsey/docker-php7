FROM php:7.4

MAINTAINER Joshua Gigg <giggsey@gmail.com>

ENV XDEBUG_VERSION 2.9.1

ENV MEMCACHE_REVISION '09e21a6'

# Install additional packages
RUN apt-get update && apt-get install -y git gnupg2 msmtp-mta openssh-client software-properties-common rsync --no-install-recommends && rm -r /var/lib/apt/lists/*

RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - \
    && apt-get install -y nodejs --no-install-recommends \
    && rm -r /var/lib/apt/lists/*

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" > /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y yarn --no-install-recommends \
    && rm -r /var/lib/apt/lists/*

RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add - \
    && add-apt-repository \
          "deb [arch=amd64] https://download.docker.com/linux/debian \
          $(lsb_release -cs) \
          stable" \
    && apt-get update \
    && apt-get install -y docker-ce --no-install-recommends \
    && rm -r /var/lib/apt/lists/*

# Install php extensions
RUN buildDeps=" \
        freetds-dev \
        libbz2-dev \
        libc-client-dev \
        libenchant-dev \
        libfreetype6-dev \
        libgmp3-dev \
        libicu-dev \
        libjpeg62-turbo-dev \
        libkrb5-dev \
        libldap2-dev \
        libmcrypt-dev \
        libmemcached-dev \
        libonig-dev \
        libpq-dev \
        libpspell-dev \
        librabbitmq-dev \
        libsasl2-dev \
        libsnmp-dev \
        libssl-dev \
        libtidy-dev \
        libxml2-dev \
        libxpm-dev \
        libxslt1-dev \
        zlib1g-dev \
    " \
    && phpModules=" \
        bcmath bz2 calendar dba enchant exif ftp gd gettext gmp imap intl ldap mbstring mysqli opcache pcntl pdo pdo_dblib pdo_mysql pdo_pgsql pgsql pspell shmop snmp soap sockets sysvmsg sysvsem sysvshm tidy xmlrpc xsl zip xdebug \
    " \
    && echo "deb http://httpredir.debian.org/debian jessie contrib non-free" > /etc/apt/sources.list.d/additional.list \
    && apt-get update && apt-get install -y libc-client2007e libpng-dev libenchant1c2a libfreetype6 libmemcached11 libmemcachedutil2 libjpeg62-turbo libmcrypt4 libpq5 libsybdb5 libx11-6 libxpm4 libzip-dev zip unzip libzip4 libxslt1.1 snmp --no-install-recommends \
    && apt-get install -y $buildDeps --no-install-recommends \
    && docker-php-source extract \
    && cd /usr/src/php/ext/ \
    && git clone git://github.com/xdebug/xdebug.git \
    && ln -s /usr/include/x86_64-linux-gnu/gmp.h /usr/include/gmp.h \
    && ln -s /usr/lib/x86_64-linux-gnu/libldap_r.so /usr/lib/libldap.so \
    && ln -s /usr/lib/x86_64-linux-gnu/libldap_r.a /usr/lib/libldap_r.a \
    && ln -s /usr/lib/x86_64-linux-gnu/libsybdb.a /usr/lib/libsybdb.a \
    && ln -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/libsybdb.so \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg-dir=/usr/include/ --with-xpm-dir=/usr/include/ \
    && PHP_OPENSSL=yes docker-php-ext-configure imap --with-imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-configure ldap --with-ldap-sasl \
    && docker-php-ext-install $phpModules \
    && printf "\n" | pecl install amqp \
    && pecl install mongodb \
    && pecl install redis \
    && for ext in $phpModules; do \
           rm -f /usr/local/etc/php/conf.d/docker-php-ext-$ext.ini; \
       done \
    && pecl install igbinary \
    # Install memcached (manually build in order to be able to enable igbinary)
    && echo no | pecl install -o --nobuild memcached \
    && cd /tmp/pear/temp/memcached/ \
    && phpize \
    && ./configure --enable-memcached-igbinary \
    && make \
    && make install \
    && cd ~ \
    && rm -rf /tmp/pear/temp/memcached/ \
    && git clone https://github.com/remicollet/pecl-memcache.git /tmp/memcache/ \
    && cd /tmp/memcache/ \
    && git reset --hard $MEMCACHE_REVISION \
    && phpize \
    && ./configure && make && make install \
    && rm -rf /tmp/memcache/ \
    && docker-php-source delete \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps

RUN apt-get update && apt-get install -y libpng-dev --no-install-recommends \
    && rm -r /var/lib/apt/lists/*

# Install composer and put binary into $PATH
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/ \
    && ln -s /usr/local/bin/composer.phar /usr/local/bin/composer

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
CMD ["php", "-a"]

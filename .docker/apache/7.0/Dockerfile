FROM alpine:3.6
MAINTAINER tess@ten7.com

# Create a apache user with the home directory.
RUN adduser -g apache -D -h /var/www apache && \
    mkdir -p -m 774 /var/www && \
    chown apache:apache -R /var/www

# Update all the Alpine packages.
RUN apk -U upgrade

# Install Apache and PHP.
RUN apk add --update --no-cache \
            ansible \
            py-mysqldb \
            bash \
            libcap \
            zlib \
            apache2 \
            apache2-ssl \
            php7-apache2 \
            php7 \
            git \
            patch \
            curl \
            php7-ctype \
            php7-curl \
            php7-dom \
            php7-gd \
            php7-iconv \
            php7-json \
            php7-ldap \
            php7-mbstring \
            php7-mcrypt \
            php7-memcached \
            php7-mysqlnd \
            php7-opcache \
            php7-openssl \
            php7-phar \
            php7-sqlite3 \
            php7-pdo \
            php7-pdo_mysql \
            php7-pdo_sqlite \
            php7-phar \
            php7-soap \
            php7-session \
            php7-simplexml \
            php7-tokenizer \
            php7-xdebug \
            php7-exif \
            php7-xml \
            php7-xmlwriter \
            php7-zip \
            php7-zlib \
            mariadb-client \
            rsync \
            openssh-client \
            nodejs-npm \
    && rm -rf /tmp/* \
    && rm -rf /var/cache/apk/*

# Allow Apache to allocate ports as non-root.
RUN setcap cap_net_bind_service=+ep /usr/sbin/httpd

# Chown the /etc/mysql directory so we can change my.cnf settings.
RUN mkdir -p -m 774 \
      /run/apache2 \
      /etc/apache2/sites.d && \
    chown -R apache:apache \
      /etc/apache2 \
      /etc/apache2/sites.d \
      /etc/php7 \
      /run/apache2 \
      /var/log/apache2 \
      /var/www

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/apache2/access.log && \
    ln -sf /dev/stderr /var/log/apache2/error.log && \
    ln -sf /dev/stdout /var/log/apache2/000_default-access_log && \
    ln -sf /dev/stderr /var/log/apache2/000_default-error_log && \
    ln -sf /dev/stdout /var/log/apache2/000_default_ssl-access_log && \
    ln -sf /dev/stderr /var/log/apache2/000_default_ssl-error_log

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

# Install drush
ADD https://github.com/drush-ops/drush/archive/8.1.16.tar.gz /tmp/drush.tar.gz
RUN mkdir -p -m 776 /usr/local/share/drush && \
    tar --strip-components=1 -xzf /tmp/drush.tar.gz -C /usr/local/share/drush && \
    cd /usr/local/share/drush && \
    composer install --prefer-dist --no-interaction && \
    chmod -R a+xr /usr/local/share/drush && \
    ln -sfn /usr/local/share/drush/drush /usr/local/bin/drush

# Install Drupal console
RUN curl https://drupalconsole.com/installer -L -o /usr/bin/drupal && \
    chmod 777 /usr/bin/drupal

# Install sass. We uninstall build dependencies afterwards to save space.
RUN apk add --update build-base libffi-dev ruby ruby-dev \
        && gem install sass compass sass-globbing --no-ri --no-rdoc \
        && apk del build-base libffi-dev ruby-dev \
        && rm -rf /var/cache/apk/*

# Copy the Apache configuration files.
COPY httpd.conf /etc/apache2/httpd.conf
COPY 000_default.conf /etc/apache2/sites.d/
COPY ssl.conf /etc/apache2/conf.d/ssl.conf

# Change the ownership of the Apache SSL files.
RUN chown -R apache:www-data /etc/ssl/apache2

# Copy PHP configuration files.
COPY php.ini /etc/php7/
COPY 00_opcache.ini /etc/php7/conf.d/00_opcache.ini
COPY xdebug.ini /etc/php7/conf.d/xdebug.ini

# Copy Drush configuration files.
COPY drushrc.php /etc/drush/
COPY aliases.drushrc.php /etc/drush/

# Copy various key scripts to a directory in $PATH.
COPY docker-entrypoint.sh /usr/local/bin/
COPY vim.sh /usr/local/bin/vim

# Make them all executable.
RUN chmod a+x /usr/local/bin/docker-entrypoint.sh && \
    chmod a+x /usr/local/bin/vim

# Copy the entire contents of the init.d directory.
COPY docker-entrypoint-init.d /docker-entrypoint-init.d
RUN chmod a+x -R /docker-entrypoint-init.d

# Copy the root bashrc file.
COPY bashrc-root /root/.bashrc

EXPOSE 80 443
WORKDIR /var/www/html
USER apache

# Copy the user bashrc file.
COPY bashrc-user /var/www/.bashrc

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND", "-f", "/etc/apache2/httpd.conf"]

FROM php:8.4.3-fpm

RUN apt-get update

RUN apt-get install -y libpq-dev unzip libaio1 libaio-dev curl \
  && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
  && docker-php-ext-install pdo pdo_pgsql \
  && pecl install redis \
  && docker-php-ext-enable redis

ENV NGINX_USER="nginx" \
    NGINXR_UID="101" \
    NGINX_GROUP="nginx" \
    NGINX_GID="101"

RUN set -ex; \
  groupadd -r --gid "$NGINX_GID" "$NGINX_GROUP"; \
  useradd -r --uid "$NGINXR_UID" --gid "$NGINX_GID" "$NGINX_USER"

RUN mkdir -p /opt/oracle

ADD docker/php/instantclient-basic-linux.x64-11.2.0.4.0.zip /opt/oracle
ADD docker/php/instantclient-sdk-linux.x64-11.2.0.4.0.zip /opt/oracle
ADD docker/php/instantclient-sqlplus-linux.x64-11.2.0.4.0.zip /opt/oracle

RUN cd /opt/oracle && unzip instantclient-basic-linux.x64-11.2.0.4.0.zip && rm instantclient-basic-linux.x64-11.2.0.4.0.zip
RUN cd /opt/oracle && unzip instantclient-sdk-linux.x64-11.2.0.4.0.zip && rm instantclient-sdk-linux.x64-11.2.0.4.0.zip
RUN cd /opt/oracle && unzip instantclient-sqlplus-linux.x64-11.2.0.4.0.zip && rm instantclient-sqlplus-linux.x64-11.2.0.4.0.zip

RUN cd /opt/oracle/instantclient_11_2 && ln -s /opt/oracle/instantclient_11_2/libclntsh.so.11.1 libclntsh.so
RUN cd /opt/oracle/instantclient_11_2 && ln -s /opt/oracle/instantclient_11_2/libocci.so.11.1 libocci.so

RUN echo /opt/oracle/instantclient_11_2 > /etc/ld.so.conf.d/oracle-instantclient.conf && ldconfig

ENV LD_LIBRARY_PATH /opt/oracle/instantclient_11_2

# Install Oracle extensions
RUN echo 'instantclient,/opt/oracle/instantclient_11_2' | pecl install pdo_oci \
    && docker-php-ext-enable pdo_oci

RUN echo 'instantclient,/opt/oracle/instantclient_11_2' | pecl install oci8 \
    && docker-php-ext-enable oci8

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- \
        --filename=composer \
        --install-dir=/usr/local/bin  \
    && composer

RUN apt-get install -y supervisor

RUN apt-get install -y npm nodejs

RUN chown -R 33:33 /var/www
WORKDIR /var/www/html/htdocs

COPY docker/php-fpm/zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf
COPY docker/php/laravel-worker.conf /etc/supervisor/conf.d/laravel-worker.conf

RUN touch /var/log/supervisor/supervisord.log

CMD ["/usr/bin/supervisord", "-n",  "-c", "/etc/supervisor/supervisord.conf"]

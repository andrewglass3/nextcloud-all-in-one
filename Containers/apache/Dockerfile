# Caddy is a requirement
FROM caddy:2.5.0-alpine as caddy

FROM debian:bullseye-20220418-slim

EXPOSE 80

RUN mkdir -p /mnt/data; \
    chown www-data:www-data /mnt/data;

VOLUME /mnt/data

RUN set -ex; \
    \
    apt-get update; \
    apt-get install -y --no-install-recommends \
        apache2 \
        supervisor \
        wget \
        ca-certificates \
        openssl \
        netcat \
        dpkg-dev \
    ; \
    rm -rf /var/lib/apt/lists/*

COPY --from=caddy /usr/bin/caddy /usr/bin/
RUN chmod +x /usr/bin/caddy

RUN a2enmod rewrite \
    headers \
    proxy \
    proxy_fcgi \
    setenvif \
    env \
    mime \
    dir \
    authz_core \
    alias

COPY nextcloud.conf /etc/apache2/sites-available/

RUN set -ex; \
    a2dissite 000-default && \
    a2dissite default-ssl && \
    a2ensite nextcloud.conf && \
    rm -rf /var/www/html/* && \
    service apache2 restart; \
    chown www-data:www-data -R /var/log/apache2; \
    chown -R www-data:www-data /var/run/apache2; \
    chown -R www-data:www-data /var/www;

RUN mkdir /var/log/supervisord; \
    mkdir /var/run/supervisord; \
    chown www-data:www-data /var/run/supervisord; \
    chown www-data:www-data /var/log/supervisord;

COPY Caddyfile /

COPY start.sh /usr/bin/
COPY supervisord.conf /
RUN chmod +x /usr/bin/start.sh; \
    chmod +r /supervisord.conf; \
    chmod a+w /Caddyfile; \
    chmod 777 /; \
    chmod +r -R /etc/apache2

# Give root a random password
RUN echo "root:$(openssl rand -base64 12)" | chpasswd

USER www-data

ENTRYPOINT ["start.sh"]
CMD ["/usr/bin/supervisord", "-c", "/supervisord.conf"]
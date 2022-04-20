FROM nginx:alpine

RUN apk update && \
    apk upgrade && \
    apk add --no-cache openssl logrotate

RUN set -x ; \
    addgroup -g 1000 -S www-data ; \
    adduser -u 1000 -D -S -G www-data www-data && exit 0 ; exit 1

RUN rm -rf /etc/nginx/nginx.conf && \
    chown -R www-data:www-data /var/log/nginx && \
    mkdir -p \
            /etc/nginx/addon.d \
            /etc/nginx/conf.d \
            /etc/nginx/hosts.d

RUN touch /var/log/messages
COPY ./modules/nginx/logrotate/nginx /etc/logrotate.d/

COPY ./modules/nginx/keys/ /etc/ssl/keys
COPY ./modules/nginx/configs/ /etc/nginx

CMD ["nginx", "-g", "daemon off;"]

# Metadata
LABEL org.opencontainers.image.vendor="Maxim Danileico" \
    org.opencontainers.image.title="Nginx"
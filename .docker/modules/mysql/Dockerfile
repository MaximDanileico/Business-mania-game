FROM mysql:8

ARG TIMEZONE=UTC
ENV TZ ${TIMEZONE}

RUN ln -snf /usr/share/zoneinfo/$TIMEZONE /etc/localtime && \
    echo $TIMEZONE > /etc/timezone && \
    chown -R mysql:root /var/lib/mysql/

COPY ./modules/mysql/conf.d/custome.cnf /etc/mysql/conf.d/custome.cnf
RUN chmod 0444 /etc/mysql/conf.d/custome.cnf

CMD ["mysqld"]

# Metadata
LABEL org.opencontainers.image.vendor="Maxim Danileico" \
    org.opencontainers.image.title="MySQL v8" \
    org.opencontainers.image.version="8"
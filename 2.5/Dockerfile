FROM debian:bullseye
MAINTAINER Agus Syafaat <agus@severalnines.com>

ENV VERSION 2.5.2

RUN apt-get update && \
    apt-get install -y wget default-mysql-client inotify-tools procps && \
    wget https://github.com/sysown/proxysql/releases/download/v${VERSION}/proxysql_${VERSION}-debian11_amd64.deb -O /opt/proxysql_${VERSION}-debian11_amd64.deb && \
    dpkg -i /opt/proxysql_${VERSION}-debian11_amd64.deb && \
    rm -f /opt/proxysql_${VERSION}-debian11_amd64.deb && \
    rm -rf /var/lib/apt/lists/*

VOLUME /var/lib/proxysql
EXPOSE 6032 6033 6080

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

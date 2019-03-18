FROM debian:stretch
MAINTAINER Ashraf Sharif <ashraf@severalnines.com>

RUN apt-get update && \
    apt-get install -y curl wget mysql-client inotify-tools procps && \
    curl -s https://api.github.com/repos/sysown/proxysql/releases/latest \
    | grep browser_download_url \
    | grep debian9 \
    | cut -d '"' -f 4 \
    | grep -Ev "clickhouse|dbg" \
    | wget -O /opt/proxysql_latest-debian9_amd64.deb -i - && \
    dpkg -i /opt/proxysql_latest-debian9_amd64.deb && \
    rm -f /opt/proxysql_latest-debian9_amd64.deb && \
    rm -rf /var/lib/apt/lists/*

VOLUME /var/lib/proxysql
EXPOSE 6032 6033

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]

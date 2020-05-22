# ProxySQL Docker Image #

## Supported Tags ##

* [2.0, 2.0.12, latest (2.0/Dockerfile)](https://github.com/severalnines/proxysql-docker/blob/master/2.0/Dockerfile)
* [1.4, 1.4.16 (1.4/Dockerfile)](https://github.com/severalnines/proxysql-docker/blob/master/1.4/Dockerfile)


## Overview ##

ProxySQL is a high-performance SQL proxy. Details at [ProxySQL](http://www.proxysql.com/) website.

## Image Description ##

This image is maintained by Severalnines and will be updated regularly on best-effort basis. The image is based on Debian 9 (Stretch) and consists of:
* mysql client
* ProxySQL package for Debian 9

## Run ##

To run a ProxySQL container with a custom ProxySQL configuration file:
```bash
$ docker run -d \
--name proxysql2 \
--publish 6033:6033 \
--publish 6032:6032 \
--publish 6080:6080 \
--restart=unless-stopped \
-v /root/proxysql/proxysql.cnf:/etc/proxysql.cnf \
severalnines/proxysql:2.0
```

For a list of available Docker image version, please refer to [Supported Tags](#supported-tags) section.

## Example Configurations, proxysql.cnf ##

For Kubernetes, check out the following blog posts for explanation and examples:
* [MySQL on Docker: Running ProxySQL as a Helper Container on Kubernetes](https://severalnines.com/blog/mysql-docker-running-proxysql-helper-container-kubernetes)
* [MySQL on Docker: Running ProxySQL as Kubernetes Service](https://severalnines.com/blog/mysql-docker-running-proxysql-kubernetes-service)
* [MySQL on Docker: ProxySQL Native Clustering with Kubernetes](https://severalnines.com/database-blog/mysql-docker-proxysql-native-clustering-kubernetes)

Here are some examples of ProxySQL configuration:
- [Galera (ProxySQL 2.x) with web stats port 6080](#galera-proxysql-2x-with-web-stats-port-6080)
- [Galera (ProxySQL 1.x)](#galera-proxysql-1x)
- [MySQL Replication](#mysql-replication)

For greater detail and explanation, check out [How to Run and Configure ProxySQL 2.0 for MySQL Galera Cluster on Docker](https://severalnines.com/blog/how-run-and-configure-proxysql-20-mysql-galera-cluster-docker).

You can also find some examples of proxysql.cnf in the official repository: https://github.com/sysown/proxysql/blob/master/doc/configuration.md

### Galera (ProxySQL 2.x) with web stats port 6080 ###

```bash
datadir="/var/lib/proxysql"

admin_variables=
{
    admin_credentials="admin:admin"
    mysql_ifaces="0.0.0.0:6032"
    refresh_interval=2000
    web_enabled=true
    web_port=6080
    stats_credentials="stats:admin"
}

mysql_variables=
{
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033;/tmp/proxysql.sock"
    default_schema="information_schema"
    stacksize=1048576
    server_version="5.1.30"
    connect_timeout_server=10000
    monitor_history=60000
    monitor_connect_interval=200000
    monitor_ping_interval=200000
    ping_interval_server_msec=10000
    ping_timeout_server=200
    commands_stats=true
    sessions_sort=true
    monitor_username="proxysql"
    monitor_password="proxysqlpassword"
    monitor_galera_healthcheck_interval=2000
    monitor_galera_healthcheck_timeout=800
}

mysql_galera_hostgroups =
(
    {
        writer_hostgroup=10
        backup_writer_hostgroup=20
        reader_hostgroup=30
        offline_hostgroup=9999
        max_writers=1
        writer_is_also_reader=2
        max_transactions_behind=30
        active=1
    }
)

mysql_servers =
(
    { address="db1.cluster.local" , port=3306 , hostgroup=10, max_connections=100 },
    { address="db2.cluster.local" , port=3306 , hostgroup=10, max_connections=100 },
    { address="db3.cluster.local" , port=3306 , hostgroup=10, max_connections=100 }
)

mysql_query_rules =
(
    {
        rule_id=100
        active=1
        match_pattern="^SELECT .* FOR UPDATE"
        destination_hostgroup=10
        apply=1
    },
    {
        rule_id=200
        active=1
        match_pattern="^SELECT .*"
        destination_hostgroup=20
        apply=1
    },
    {
        rule_id=300
        active=1
        match_pattern=".*"
        destination_hostgroup=10
        apply=1
    }
)

mysql_users =
(
    { username = "wordpress", password = "passw0rd", default_hostgroup = 10, transaction_persistent = 0, active = 1 },
    { username = "sbtest", password = "passw0rd", default_hostgroup = 10, transaction_persistent = 0, active = 1 }
)
```

### Galera (ProxySQL 1.x) ###

```bash
datadir="/var/lib/proxysql"

admin_variables=
{
    admin_credentials="admin:admin"
    mysql_ifaces="0.0.0.0:6032"
    refresh_interval=2000
}

mysql_variables=
{
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033;/tmp/proxysql.sock"
    default_schema="information_schema"
    stacksize=1048576
    server_version="5.1.30"
    connect_timeout_server=10000
    monitor_history=60000
    monitor_connect_interval=200000
    monitor_ping_interval=200000
    ping_interval_server_msec=10000
    ping_timeout_server=200
    commands_stats=true
    sessions_sort=true
    monitor_username="proxysql"
    monitor_password="proxysqlpassword"
}

mysql_servers =
(
    { address="db1.cluster.local" , port=3306 , hostgroup=10, max_connections=100 },
    { address="db2.cluster.local" , port=3306 , hostgroup=10, max_connections=100 },
    { address="db3.cluster.local" , port=3306 , hostgroup=10, max_connections=100 },
    { address="db1.cluster.local" , port=3306 , hostgroup=20, max_connections=100 },
    { address="db2.cluster.local" , port=3306 , hostgroup=20, max_connections=100 },
    { address="db3.cluster.local" , port=3306 , hostgroup=20, max_connections=100 }
)

mysql_query_rules =
(
    {
        rule_id=100
        active=1
        match_pattern="^SELECT .* FOR UPDATE"
        destination_hostgroup=10
        apply=1
    },
    {
        rule_id=200
        active=1
        match_pattern="^SELECT .*"
        destination_hostgroup=20
        apply=1
    },
    {
        rule_id=300
        active=1
        match_pattern=".*"
        destination_hostgroup=10
        apply=1
    }
)

scheduler =
(
    {
        id = 1
        filename = "/usr/share/proxysql/tools/proxysql_galera_checker.sh"
        active = 1
        interval_ms = 2000
        arg1 = "10"
        arg2 = "20"
        arg3 = "1"
        arg4 = "1"
        arg5 = "/var/lib/proxysql/proxysql_galera_checker.log"
    }
)

mysql_users =
(
    { username = "sbtest" , password = "passw0rd" , default_hostgroup = 10 , active = 1 }
)

```

### MySQL Replication ###

```bash
datadir="/var/lib/proxysql"

admin_variables=
{
    admin_credentials="admin:admin"
    mysql_ifaces="0.0.0.0:6032"
    refresh_interval=2000
}

mysql_variables=
{
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033;/tmp/proxysql.sock"
    default_schema="information_schema"
    stacksize=1048576
    server_version="5.1.30"
    connect_timeout_server=10000
    monitor_history=60000
    monitor_connect_interval=200000
    monitor_ping_interval=200000
    ping_interval_server_msec=10000
    ping_timeout_server=200
    commands_stats=true
    sessions_sort=true
    monitor_username="proxysql"
    monitor_password="proxysqlpassword"
}

mysql_replication_hostgroups =
(
    { writer_hostgroup=10 , reader_hostgroup=20 , comment="host groups" }
)

mysql_servers =
(
    { address="master.replication.local" , port=3306 , hostgroup=10, max_connections=100 , max_replication_lag = 5 },
    { address="slave1.replication.local" , port=3306 , hostgroup=20, max_connections=100 , max_replication_lag = 5 },
    { address="slave2.replication.local" , port=3306 , hostgroup=20, max_connections=100 , max_replication_lag = 5 }
)

mysql_query_rules =
(
    {
        rule_id=100
        active=1
        match_pattern="^SELECT .* FOR UPDATE"
        destination_hostgroup=10
        apply=1
    },
    {
        rule_id=200
        active=1
        match_pattern="^SELECT .*"
        destination_hostgroup=20
        apply=1
    },
    {
        rule_id=300
        active=1
        match_pattern=".*"
        destination_hostgroup=10
        apply=1
    }
)

mysql_users =
(
    { username = "sbtest" , password = "password" , default_hostgroup = 10 , active = 1 }
)
```

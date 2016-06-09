# pgpoolcentos
pgpool on centos

### Centos 7 INstallation

```sh
# yum install http://www.pgpool.net/yum/rpms/3.4/redhat/rhel-7-x86_64/pgpool-II-release-3.4-1.noarch.rpm
# yum install pgpool-II-pg93 pgpool-II-pg93-extensions
or
# yum install pgpool 

```


### Instalation

```sh
# cd /etc/pgpool-II
generate your md5 user password with pg_md5 and add aoutomatically on pool_passwd
# pg_md5 -u postgres -m -p
md5a13f595e93dc71bef638a3a4a2d5371f
# vim pcp.conf
pguser:md5a13f595e93dc71bef638a3a4a2d5371f
# vim pgpool.conf
# - Backend Connection Settings -

backend_hostname0 = 'localhost'
                                   # Host name or IP address to connect to for backend 0
backend_port0 = 5432
                                   # Port number for backend 0
backend_weight0 = 1
                                   # Weight for backend 0 (only in load balancing mode)
#backend_data_directory0 = '/var/lib/pgsql/data'
                                   # Data directory for backend 0
backend_flag0 = 'ALLOW_TO_FAILOVER'
                                   # Controls various backend behavior
                                   # ALLOW_TO_FAILOVER or DISALLOW_TO_FAILOVER
backend_hostname1 = 'belant.ddns.net'
backend_port1 = 5432
backend_weight1 = 1
#backend_data_directory1 = '/data1'
backend_flag1 = 'ALLOW_TO_FAILOVER'

backend_hostname2 = '10.200.0.222'
backend_port2 = 5432
backend_weight2 = 1
backend_flag2 = 'ALLOW_TO_FAILOVER'

backend_hostname3 = '10.200.0.223'
backend_port3 = 5432
backend_weight3 = 1
backend_flag3 = 'ALLOW_TO_FAILOVER'

# - Authentication -

enable_pool_hba = on
                                   # Use pool_hba.conf for client authentication
pool_passwd = 'pool_passwd'
                                   # File name of pool_passwd for md5 authentication.
                                   # "" disables pool_passwd.
                                   # (change requires restart)
authentication_timeout = 60
                                   # Delay in seconds to complete client authentication
                                   # 0 means no timeout.

load_balance_mode = on
replication_mode = o

# vim pool_hba.conf
# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD

# "local" is for Unix domain socket connections only
local   all         all                               md5
# IPv4 local connections:
host    all         all         10.200.0.0/24          md5
host    all         all         127.0.0.1/32          md5
host    all         all         ::1/128               md5

# mkdir /var/run/pgpool
# systemctl start pgpool
# psql -h 10.200.0.251 -U pguser -p 9999
```

### psql command
```sh
pguser=# create DATABASE simpon;
pguser=# \c simpon;
pguser=# CREATE TABLE DEPARTMENT(
   ID INT PRIMARY KEY      NOT NULL,
   DEPT           CHAR(50) NOT NULL,
   EMP_ID         INT      NOT NULL
);
pguser=# INSERT INTO DEPARTMENT VALUES (1,'koplak',3);
pguser=# UPDATE DEPARTMENT SET DEPT = 'cuukkk' WHERE ID = 1;
pguser=# DELETE FROM DEPARTMENT WHERE ID = 1;
pguser=# CREATE SCHEMA transportasi;
pguser=# CREATE TABLE transportasi.DEPARTMENT(
   ID INT PRIMARY KEY      NOT NULL,
   DEPT           CHAR(50) NOT NULL,
   EMP_ID         INT      NOT NULL
);
pguser=# INSERT INTO transportasi.DEPARTMENT VALUES (1,'koplak',3);
pguser=# UPDATE transportasi.DEPARTMENT SET DEPT = 'cuukkk' WHERE ID = 1;
pguser=# DELETE FROM transportasi.DEPARTMENT WHERE ID = 1;
```

### Bug on PGPool
Please pay attention, sometime if we shutdown or reboot our machine when pgpool its not responding, it will be block the port in the setting. So change port and start again. You may considere to use haproxy to connect used port to the new pgpool port(e.g. 9999).

```sh /etc/haproxy/haproxy.cfg
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
#    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen stats :1936
    mode http
    stats enable
    stats hide-version
    stats realm Haproxy\ Statistics
    stats uri /haproxy?stats
    stats auth admin:nimda2016

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  main *:5000
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js

    use_backend static          if url_static
    default_backend             app

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server  app1 127.0.0.1:5001 check
    server  app2 127.0.0.1:5002 check
    server  app3 127.0.0.1:5003 check
    server  app4 127.0.0.1:5004 check

listen pgsql_cluster 0.0.0.0:5432
    mode tcp
    option tcplog
    balance roundrobin
    server db01 127.0.0.1:9999 check
    server db01 10.200.0.203:9999 check
#    server db02 10.200.0.221:5432 check
#    server db03 10.200.0.222:5432 check
#    server db04 10.200.0.223:5432 check


``` 


### Reference

[pgpool-II Tutorial]http://www.pgpool.net/docs/latest/tutorial-en.html

[psql tutorial]http://www.tutorialspoint.com/postgresql/postgresql_schema.htm
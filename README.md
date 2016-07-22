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
replication_mode = 1




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
Please pay attention, do not use restart for service, please use stop and then wait for several minutes you can start the service. 
sometime if we reboot our machine when pgpool its not responding, it will be block the port in the setting. So change port and start again. You may considere to use port forwarding to connect used port to the new pgpool port(e.g. 9999).
For debugging purpose use this command : 
_pgpool -n -d_

```sh
# firewall-cmd --zone=public --add-masquerade --permanent
# firewall-cmd --zone=public --add-forward-port=port=5432:proto=tcp:toport=9999 --permanent
# firewall-cmd --reload
# firewall-cmd --zone=public --list-all
```

### Setting max connection
num_init_children :concurrent connections limit to pgpool-II from clients

num_init_children*listen_backlog_multiplier exceeds the number, you need to set the backlong higher
netstat -s
# sysctl net.core.somaxconn
net.core.somaxconn = 128
 /etc/sysctl.conf
# sysctl -w net.core.somaxconn=256
net.core.somaxconn = 256

max_pool
The maximum number of cached connections in pgpool-II children processes. pgpool-II reuses the cached connection if an incoming connection is connecting to the same database with the same user name. If not, pgpool-II creates a new connection to the backend. If the number of cached connections exceeds max_pool, the oldest connection will be discarded, and uses that slot for the new connection.
*Default value is 4. Please be aware that the number of connections from pgpool-II processes to the backends may reach num_init_children * max_pool.*
This parameter can only be set at server start.


### Reference

[pgpool-II Tutorial]http://www.pgpool.net/docs/latest/tutorial-en.html

[PGPool Docs]http://www.pgpool.net/docs/latest/pgpool-en.html

[psql tutorial]http://www.tutorialspoint.com/postgresql/postgresql_schema.htm
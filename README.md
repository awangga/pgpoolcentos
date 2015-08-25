# pgpoolcentos
pgpool on centos


### INstalation
```sh
# yum install pgpool
# cd /etc/pgpool-II
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

# vim pool_hba.conf
# TYPE  DATABASE    USER        CIDR-ADDRESS          METHOD

# "local" is for Unix domain socket connections only
local   all         all                               md5
# IPv4 local connections:
host    all         all         127.0.0.1/32          md5
host    all         all         ::1/128               md5
# mv pool_passwd pool_passwd.old
# cp pcp.conf pool_passwd
# psql -U pguser -p 9999
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
```

### Reference

[pgpool-II Tutorial]http://www.pgpool.net/docs/latest/tutorial-en.html
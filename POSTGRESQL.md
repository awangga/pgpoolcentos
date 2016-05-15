# Postgre Instalation on Centos 7
Instalation on centos 7


### Instalation PostgreSQL 9.2

```sh
# yum install postgresql-server postgresql-contrib
# postgresql-setup initdb
# vi /var/lib/pgsql/data/pg_hba.conf
IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# su postgres
$ postgres -D /var/lib/pgsql/data
$ pg_ctl -D /var/lib/pgsql/data -l logfile start

```
### Remove Installation

```sh
# yum remove postgresql-server postgresql-contrib postgresql postgresql-libs 
```

### Instalation PostgreSQL 9.3

```sh
# rpm -iUvh http://yum.postgresql.org/9.3/redhat/rhel-7-x86_64/pgdg-centos93-9.3-1.noarch.rpm
# yum -y update
# yum -y install postgresql93 postgresql93-server postgresql93-contrib postgresql93-libs
# systemctl enable postgresql-9.3
# /usr/pgsql-9.3/bin/postgresql93-setup initdb
# vi /var/lib/pgsql/9.3/data/pg_hba.conf
IPv4 local connections:
host    all             all             127.0.0.1/32            md5
host    replication     postgres        10.200.0.0/24            md5
# vi /var/lib/pgsql/9.3/data/postgresql.conf 
listen_addresses='*'
# systemctl start postgresql-9.3
# su postgres
$ psql
```

### Reference

[Instalation]https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-centos-7

[9.3]http://www.liquidweb.com/kb/how-to-install-and-connect-to-postgresql-on-centos-7/
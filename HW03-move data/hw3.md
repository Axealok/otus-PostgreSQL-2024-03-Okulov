# Перенос содержимого базы данных PostgreSQL
## Установка тестового полигона

```
C:\Users\okulovan>ssh -i C:/Users/okulovan/OkulovAN okulovan@176.109.103.34
$ sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt$(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
$ sudo -u postgres psql -p 5432
postgres=# ALTER USER postgres PASSWORD 'postgres';
ALTER ROLE
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# insert into test values('TestValue');
INSERT 0 1
postgres=# select * from test
postgres-# ;
    c1
-----------
 TestValue
(1 row)

postgres=#
```

## Подключение дополнительного диска
```
$ sudo -u postgres pg_ctlcluster 15 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@15-main
$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```

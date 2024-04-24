# Работа с базами данных, пользователями и правами

```
$ sudo apt install postgresql-14
$ pg_lsclusters
Ver Cluster Port Status Owner    Data directory              Log file
14  main    5432 online postgres /var/lib/postgresql/14/main /var/log/postgresql/postgresql-14-main.log
$ sudo -u postgres psql -p 5432
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))

postgres=# create database testdb;
CREATE DATABASE
postgres=# \c testdb
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# create schema testnm;
CREATE SCHEMA
testdb=# create table t1(i int)
testdb-# ;
CREATE TABLE
testdb=# insert into t1 values(1)
testdb-# ;
INSERT 0 1
testdb=# select * from t1;
 i 
---
 1
(1 строка)


```

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
testdb=# create role readonly;
CREATE ROLE
testdb=# grant connect on database testdb to readonly;
GRANT
testdb=# grant usage on schema testnm to readonly
testdb-# ;
GRANT
testdb=# grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
GRANT
testdb=# CREATE USER testread with password 'test123';
CREATE ROLE
testdb=# grant readonly TO testread;
GRANT ROLE
testdb=# \c testdb testread
подключиться к серверу через сокет "/var/run/postgresql/.s.PGSQL.5432" не удалось: ВАЖНО:  пользователь "testread" не прошёл проверку подлинности (Peer)
Сохранено предыдущее подключение
```
Ошибка из-за метода аутентификации. Либо меняем hba  либо явно указываем в строке подключения, как у меня далее
```
testdb=# \q
root@b1-t-oan:/etc/postgresql/14/main# sudo -u postgres psql -U testread -h 127.0.0.1 -W -d postgres
Пароль: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
SSL-соединение (протокол: TLSv1.3, шифр: TLS_AES_256_GCM_SHA384, сжатие: выкл.)
Введите "help", чтобы получить справку.

postgres=> \c testdb testread
Пароль пользователя testread: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
SSL-соединение (протокол: TLSv1.3, шифр: TLS_AES_256_GCM_SHA384, сжатие: выкл.)
Вы подключены к базе данных "testdb" как пользователь "testread".
testdb=> select * from t1;
ОШИБКА:  нет доступа к таблице t1
testdb=> /dt
testdb-> ;
ОШИБКА:  ошибка синтаксиса (примерное положение: "/")
СТРОКА 1: /dt
          ^
testdb=> \dt
         Список отношений
 Схема  | Имя |   Тип   | Владелец 
--------+-----+---------+----------
 public | t1  | таблица | postgres
(1 строка)
```
Ошибка в том что таблица создана в схеме public  а права раздавали на схему  testnm
```
testdb=> \q
root@b1-t-oan:/etc/postgresql/14/main# sudo -u postgres psql -p 5432
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
Введите "help", чтобы получить справку.

postgres=# DROP TABLE t1;
ОШИБКА:  таблица "t1" не существует
postgres=# \c testdb
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# \dt
         Список отношений
 Схема  | Имя |   Тип   | Владелец 
--------+-----+---------+----------
 public | t1  | таблица | postgres
(1 строка)

testdb=# DROP TABLE t1;
DROP TABLE
testdb=# CREATE TABLE testnm.t1(c1 integer);
CREATE TABLE
testdb=# INSERT INTO testnm.t1 values(1);
INSERT 0 1
testdb=#
testdb=# select * from testnm.t1;
 c1 
----
  1
(1 строка)

root@b1-t-oan:/etc/postgresql/14/main# sudo -u postgres psql -U testread -h 127.0.0.1 -W -d postgres
Пароль: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
SSL-соединение (протокол: TLSv1.3, шифр: TLS_AES_256_GCM_SHA384, сжатие: выкл.)
Введите "help", чтобы получить справку.

postgres=> \c testdb
Пароль: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
SSL-соединение (протокол: TLSv1.3, шифр: TLS_AES_256_GCM_SHA384, сжатие: выкл.)
Вы подключены к базе данных "testdb" как пользователь "testread".
testdb=> select * from testnm.t1;
ОШИБКА:  нет доступа к таблице t1
```
Ошибка в том что права  SELECT назначались на существующие таблицы, а я пересоздал таблицу
Переназначаем права
```
testdb=> \q
root@b1-t-oan:/etc/postgresql/14/main# sudo -u postgres psql -p 5432
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
Введите "help", чтобы получить справку.

postgres=# \c testdb
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
Вы подключены к базе данных "testdb" как пользователь "postgres".
testdb=# grant SELECT on all TABLEs in SCHEMA testnm TO readonly;
GRANT
testdb=# \q
root@b1-t-oan:/etc/postgresql/14/main# sudo -u postgres psql -U testread -h 127.0.0.1 -W -d postgres
Пароль: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
SSL-соединение (протокол: TLSv1.3, шифр: TLS_AES_256_GCM_SHA384, сжатие: выкл.)
Введите "help", чтобы получить справку.

postgres=> \c testdb
Пароль: 
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 14.11 (Ubuntu 14.11-1.pgdg22.04+1))
SSL-соединение (протокол: TLSv1.3, шифр: TLS_AES_256_GCM_SHA384, сжатие: выкл.)
Вы подключены к базе данных "testdb" как пользователь "testread".
testdb=> select * from testnm.t1;
 c1 
----
  1
(1 строка)

```
Создаем вторую таблицу. Это нам дает сделать -так как пользователю по умолчанию разрешено создавать таблицы в схеме public
```
testdb=> 
testdb=> create table t2(c1 integer); 
CREATE TABLE
testdb=> insert into t2 values (2);
INSERT 0 1
testdb=> 
```

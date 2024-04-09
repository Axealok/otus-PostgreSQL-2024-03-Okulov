# Уровни изоляции

1-я сессия
```
postgres=# show transaction isolation level;
transaction_isolation 
-----------------------
 read committed
(1 строка)

postgres=# CREATE DATABASE test1;
CREATE DATABASE
postgres=# \c test1
psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 15.6 (Ubuntu 15.6-1.pgdg22.04+1))
Вы подключены к базе данных "test1" как пользователь "postgres".
test1=# \echo :AUTOCOMMIT
OFF
test1=# create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) >values('petr', 'petrov'); commit;
CREATE TABLE
INSERT 0 1
INSERT 0 1
COMMIT
```

2-я сессия
```
test1=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
(2 строки)
```
Так как уровень изоляции read committed - незавершенная транзакция не отображается во второй сессии

1-я сессия
```
test1=*# commit;
COMMIT
```
2-ясессия
```
test1=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)
```

Записали изменения - строка отображается

Меняем уровень изоляции и вставляем данные

```
test1=# set transaction isolation level repeatable read;
SET
test1=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 строка)

test1=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
```
Во второй сессии строка не отображается

```
test1=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
(3 строки)
```

После завершения транзакции в первой сессии во второй вижу изменения.
Проблема в том что во второй сессии не установил уровень изоляции

Устанавливаю уровень изоляции в первой и второй сессиях
Теперь после завершения транзакции в первой сессии - во второй не отображается
И толко после завершения сессии во второй сессии данные появились

1-я сессия
```
test1=*# set transaction isolation level repeatable read;
SET
test1=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 строка)

test1=*# insert into persons(first_name, second_name) values('alexey1', 'alekseev2');
INSERT 0 1
test1=*# ^C
test1=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 строка)

test1=*# insert into persons(first_name, second_name) values('alexey3', 'alekseev3');
INSERT 0 1
test1=*# commit;
COMMIT


```

2-я сессия
```
test1=*# set transaction isolation level repeatable read;
SET
test1=*# show transaction isolation level;
 transaction_isolation 
-----------------------
 repeatable read
(1 строка)

test1=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
  5 | alexey     | alekseev
  6 | alexey1    | alekseev1
(6 строк)

test1=*# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
  5 | alexey     | alekseev
  6 | alexey1    | alekseev1
(6 строк)

test1=*# commit;
COMMIT
test1=# select * from persons;
 id | first_name | second_name 
----+------------+-------------
  1 | ivan       | ivanov
  2 | petr       | petrov
  3 | sergey     | sergeev
  4 | sveta      | svetova
  5 | alexey     | alekseev
  6 | alexey1    | alekseev1
  7 | alexey1    | alekseev2
  8 | alexey3    | alekseev3
(8 строк)

test1=*# 

```

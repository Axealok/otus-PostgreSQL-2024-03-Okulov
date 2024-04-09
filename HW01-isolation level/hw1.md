1-я сессия

>postgres=# show transaction isolation level;
>transaction_isolation 
>-----------------------
> read committed
>(1 строка)
>
>postgres=# CREATE DATABASE test1;
>CREATE DATABASE
>postgres=# \c test1
>psql (16.2 (Ubuntu 16.2-1.pgdg22.04+1), сервер 15.6 (Ubuntu 15.6-1.pgdg22.04+1))
>Вы подключены к базе данных "test1" как пользователь "postgres".
>test1=# \echo :AUTOCOMMIT
>OFF
>test1=# create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) >values('petr', 'petrov'); commit;
>CREATE TABLE
>INSERT 0 1
>INSERT 0 1
>COMMIT

2-я сессия

>test1=*# select * from persons;
> id | first_name | second_name 
>----+------------+-------------
>  1 | ivan       | ivanov
>  2 | petr       | petrov
>(2 строки)

Так как уровень изоляции read committed - незавершенная транзакция не отображается во второй сессии

1-я сессия
>test1=*# commit;
>COMMIT

2-я сессия
>test1=*# select * from persons;
> id | first_name | second_name 
>----+------------+-------------
>  1 | ivan       | ivanov
>  2 | petr       | petrov
>  3 | sergey     | sergeev
>(3 строки)

Записали изменения - строка отображается




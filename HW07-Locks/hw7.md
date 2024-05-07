# Блокировки

## Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
```
locks=# ALTER SYSTEM SET log_lock_waits = on;
ALTER SYSTEM
locks=# ALTER SYSTEM SET deadlock_timeout = 200;
ALTER SYSTEM
locks=# SELECT pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 строка)

```

```
2024-05-07 11:49:50.453 +05 [349422] СООБЩЕНИЕ:  процесс 349422 продолжает ожидать в режиме ShareLock блокировку "транзакция 1099646" в течение 200.221 мс
2024-05-07 11:49:50.453 +05 [349422] ПОДРОБНОСТИ:  Process holding the lock: 349595. Wait queue: 349422.
2024-05-07 11:49:50.453 +05 [349422] КОНТЕКСТ:  при изменении кортежа (0,5) в отношении "accounts"
2024-05-07 11:49:50.453 +05 [349422] ОПЕРАТОР:  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
2024-05-07 11:50:23.123 +05 [349422] СООБЩЕНИЕ:  процесс 349422 получил в режиме ShareLock блокировку "транзакция 1099646" через 32870.332 мс
2024-05-07 11:50:23.123 +05 [349422] КОНТЕКСТ:  при изменении кортежа (0,5) в отношении "accounts"
2024-05-07 11:50:23.123 +05 [349422] ОПЕРАТОР:  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;


```

## Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах. Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. Пришлите список блокировок и объясните, что значит каждая.



## Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

1-я сессия
```
locks=# BEGIN;
BEGIN
locks=*# UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 1;
UPDATE 1

```
2-я сессия
```
locks=# BEGIN;
UPDATE accounts SET amount = amount - 10.00 WHERE acc_no = 2;
BEGIN
UPDATE 1

```
3-я сессия
```
locks=# BEGIN;
UPDATE accounts SET amount = amount - 100.00 WHERE acc_no = 3;
BEGIN
UPDATE 1

```
1-я сессия
``
locks=*# UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 2;

```
2-я сессия
```
locks=# UPDATE accounts SET amount = amount + 10.00 WHERE acc_no = 3;
BEGIN
UPDATE 1

```
3-я сессия
```
locks=*# UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
ОШИБКА:  обнаружена взаимоблокировка
ПОДРОБНОСТИ:  Процесс 349422 ожидает в режиме ShareLock блокировку "транзакция 1099651"; заблокирован процессом 349595.
Процесс 349595 ожидает в режиме ShareLock блокировку "транзакция 1099652"; заблокирован процессом 349423.
Процесс 349423 ожидает в режиме ShareLock блокировку "транзакция 1099653"; заблокирован процессом 349422.
ПОДСКАЗКА:  Подробности запроса смотрите в протоколе сервера.
КОНТЕКСТ:  при изменении кортежа (0,8) в отношении "accounts"

```

```
locks=!# rollback;
ROLLBACK
locks=# SELECT pid, wait_event_type, wait_event, pg_blocking_pids(pid)
FROM pg_stat_activity
WHERE backend_type = 'client backend' ORDER BY pid;
  pid   | wait_event_type |  wait_event   | pg_blocking_pids 
--------+-----------------+---------------+------------------
 349422 |                 |               | {}
 349423 | Client          | ClientRead    | {}
 349595 | Lock            | transactionid | {349423}
(3 rows)

locks=# CREATE VIEW locks_v AS
SELECT pid,
       locktype,
       CASE locktype
         WHEN 'relation' THEN relation::regclass::text
         WHEN 'transactionid' THEN transactionid::text
         WHEN 'tuple' THEN relation::regclass::text||':'||tuple::text
       END AS lockid,
       mode,
       granted
FROM pg_locks
WHERE locktype in ('relation','transactionid','tuple')
AND (locktype != 'relation' OR relation = 'accounts'::regclass);
CREATE VIEW
locks=# SELECT * FROM locks_v;
  pid   |   locktype    |   lockid   |       mode       | granted 
--------+---------------+------------+------------------+---------
 349423 | relation      | accounts   | RowExclusiveLock | t
 349595 | relation      | accounts   | RowExclusiveLock | t
 349595 | tuple         | accounts:9 | ExclusiveLock    | t
 349423 | transactionid | 1099652    | ExclusiveLock    | t
 349595 | transactionid | 1099651    | ExclusiveLock    | t
 349595 | transactionid | 1099652    | ShareLock        | f
(6 rows)

```

```
2024-05-07 12:10:27.384 +05 [349595] СООБЩЕНИЕ:  процесс 349595 продолжает ожидать в режиме ShareLock блокировку "транзакция 1099652" в течение 200.288 мс
2024-05-07 12:10:27.384 +05 [349595] ПОДРОБНОСТИ:  Process holding the lock: 349423. Wait queue: 349595.
2024-05-07 12:10:27.384 +05 [349595] КОНТЕКСТ:  при изменении кортежа (0,9) в отношении "accounts"
2024-05-07 12:10:27.384 +05 [349595] ОПЕРАТОР:  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 2;
2024-05-07 12:11:12.699 +05 [349423] СООБЩЕНИЕ:  процесс 349423 продолжает ожидать в режиме ShareLock блокировку "транзакция 1099653" в течение 200.185 мс
2024-05-07 12:11:12.699 +05 [349423] ПОДРОБНОСТИ:  Process holding the lock: 349422. Wait queue: 349423.
2024-05-07 12:11:12.699 +05 [349423] КОНТЕКСТ:  при изменении кортежа (0,10) в отношении "accounts"
2024-05-07 12:11:12.699 +05 [349423] ОПЕРАТОР:  UPDATE accounts SET amount = amount + 10.00 WHERE acc_no = 3;
2024-05-07 12:11:44.328 +05 [349422] СООБЩЕНИЕ:  процесс 349422 обнаружил взаимоблокировку, ожидая в режиме ShareLock блокировку "транзакция 1099651" в течение 200.155 мс
2024-05-07 12:11:44.328 +05 [349422] ПОДРОБНОСТИ:  Process holding the lock: 349595. Wait queue: .
2024-05-07 12:11:44.328 +05 [349422] КОНТЕКСТ:  при изменении кортежа (0,8) в отношении "accounts"
2024-05-07 12:11:44.328 +05 [349422] ОПЕРАТОР:  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
2024-05-07 12:11:44.328 +05 [349422] ОШИБКА:  обнаружена взаимоблокировка
2024-05-07 12:11:44.328 +05 [349422] ПОДРОБНОСТИ:  Процесс 349422 ожидает в режиме ShareLock блокировку "транзакция 1099651"; заблокирован процессом 349595.
        Процесс 349595 ожидает в режиме ShareLock блокировку "транзакция 1099652"; заблокирован процессом 349423.
        Процесс 349423 ожидает в режиме ShareLock блокировку "транзакция 1099653"; заблокирован процессом 349422.
        Процесс 349422: UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
        Процесс 349595: UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 2;
        Процесс 349423: UPDATE accounts SET amount = amount + 10.00 WHERE acc_no = 3;
2024-05-07 12:11:44.328 +05 [349422] ПОДСКАЗКА:  Подробности запроса смотрите в протоколе сервера.
2024-05-07 12:11:44.328 +05 [349422] КОНТЕКСТ:  при изменении кортежа (0,8) в отношении "accounts"
2024-05-07 12:11:44.328 +05 [349422] ОПЕРАТОР:  UPDATE accounts SET amount = amount + 100.00 WHERE acc_no = 1;
2024-05-07 12:11:44.329 +05 [349423] СООБЩЕНИЕ:  процесс 349423 получил в режиме ShareLock блокировку "транзакция 1099653" через 31830.250 мс
2024-05-07 12:11:44.329 +05 [349423] КОНТЕКСТ:  при изменении кортежа (0,10) в отношении "accounts"
2024-05-07 12:11:44.329 +05 [349423] ОПЕРАТОР:  UPDATE accounts SET amount = amount + 10.00 WHERE acc_no = 3;

```

## Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

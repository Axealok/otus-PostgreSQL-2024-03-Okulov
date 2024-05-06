# Работа с журналами
## Настраиваем создание контрольной точки каждые 30с

![Checkpoint timeout = 30s](https://github.com/Axealok/otus-PostgreSQL-2024-03-Okulov/blob/1b8f545ed16c2c5862404fc02b9a1d0b3fec2895/HW06-WAL/hw61.PNG)

Перезапускаем кластер
```
sudo pg_ctlcluster 15 main restart

```
## Запускаем нагрузочный тест
```
root@b1-t-oan:/var/lib/postgresql/15# sudo -u postgres pgbench -P 30 -T 600 -U postgres buffer_temp
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 30.0 s, 567.3 tps, lat 1.762 ms stddev 1.051, 0 failed
progress: 60.0 s, 566.2 tps, lat 1.765 ms stddev 0.828, 0 failed
progress: 90.0 s, 555.9 tps, lat 1.798 ms stddev 1.042, 0 failed
progress: 120.0 s, 560.7 tps, lat 1.783 ms stddev 0.953, 0 failed
progress: 150.0 s, 576.0 tps, lat 1.735 ms stddev 1.006, 0 failed
progress: 180.0 s, 578.4 tps, lat 1.728 ms stddev 0.772, 0 failed
progress: 210.0 s, 553.5 tps, lat 1.806 ms stddev 1.106, 0 failed
progress: 240.0 s, 600.4 tps, lat 1.665 ms stddev 0.855, 0 failed
progress: 270.0 s, 535.6 tps, lat 1.866 ms stddev 1.424, 0 failed
progress: 300.0 s, 557.1 tps, lat 1.794 ms stddev 1.010, 0 failed
progress: 330.0 s, 557.3 tps, lat 1.794 ms stddev 1.043, 0 failed
progress: 360.0 s, 578.9 tps, lat 1.727 ms stddev 1.029, 0 failed
progress: 390.0 s, 549.9 tps, lat 1.818 ms stddev 1.041, 0 failed
progress: 420.0 s, 591.6 tps, lat 1.689 ms stddev 0.686, 0 failed
progress: 450.0 s, 573.4 tps, lat 1.743 ms stddev 1.064, 0 failed
progress: 480.0 s, 605.8 tps, lat 1.650 ms stddev 0.849, 0 failed
progress: 510.0 s, 596.1 tps, lat 1.677 ms stddev 0.978, 0 failed
progress: 540.0 s, 602.0 tps, lat 1.661 ms stddev 0.816, 0 failed
progress: 570.0 s, 586.1 tps, lat 1.705 ms stddev 1.072, 0 failed
progress: 600.0 s, 582.5 tps, lat 1.716 ms stddev 0.870, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 344237
number of failed transactions: 0 (0.000%)
latency average = 1.742 ms
latency stddev = 0.985 ms
initial connection time = 7.239 ms
tps = 573.733616 (without initial connection time)


```
Смотрим информацию о кластере
```
root@b1-t-oan:/var/lib/postgresql/15# sudo /usr/lib/postgresql/15/bin/pg_controldata /var/lib/postgresql/15/main/
Номер версии pg_control:              1300
Номер версии каталога:                202209061
Идентификатор системы баз данных:     7365743920985068351
Состояние кластера БД:                в работе
Последнее обновление pg_control:      Пн 06 мая 2024 12:39:18
Положение последней конт. точки:      0/493AC290
Положение REDO последней конт. точки: 0/486B3690
Файл WAL c REDO последней к. т.:      000000010000000000000048
Линия времени последней конт. точки:  1
Пред. линия времени последней к. т.:  1
Режим full_page_writes последней к.т: вкл.
NextXID последней конт. точки:        0:1077501
NextOID последней конт. точки:        24647
NextMultiXactId послед. конт. точки:  1
NextMultiOffset послед. конт. точки:  0
oldestXID последней конт. точки:      716
БД с oldestXID последней конт. точки: 1
oldestActiveXID последней к. т.:      1077500
oldestMultiXid последней конт. точки: 1
БД с oldestMulti последней к. т.:     1
oldestCommitTsXid последней к. т.:    0
newestCommitTsXid последней к. т.:    0
Время последней контрольной точки:    Пн 06 мая 2024 12:38:51
Фиктивный LSN для нежурналир. таблиц: 0/3E8
Мин. положение конца восстановления:  0/0
Линия времени мин. положения к. в.:   0
Положение начала копии:               0/0
Положение конца копии:                0/0
Требуется запись конец-копии:         нет
Значение wal_level:                   replica
Значение wal_log_hints:               выкл.
Значение max_connections:             100
Значение max_worker_processes:        8
Значение max_wal_senders:             10
Значение max_prepared_xacts:          0
Значение max_locks_per_xact:          64
Значение track_commit_timestamp:      выкл.
Макс. предел выравнивания данных:     8
Размер блока БД:                      8192
Блоков в макс. сегменте отношений:    131072
Размер блока WAL:                     8192
Байт в сегменте WAL:                  16777216
Максимальная длина идентификаторов:   64
Макс. число столбцов в индексе:       32
Максимальный размер порции TOAST:     1996
Размер порции большого объекта:       2048
Формат хранения даты/времени:         64-битные целые
Передача аргумента float8:            по значению
Версия контрольных сумм страниц:      0
Случ. число для псевдоаутентификации: d4150dbd58ccf509c18089284ebd7942537b89d257ee29bed18f90a7364c3f8a


```
Размер WAL примерно 16Мб

```
buffer_temp=# SELECT * FROM pg_ls_waldir()
buffer_temp-# ;\
           name           |   size   |      modification      
--------------------------+----------+------------------------
 000000010000000000000048 | 16777216 | 2024-05-06 12:38:54+05
 00000001000000000000004C | 16777216 | 2024-05-06 12:38:27+05
 00000001000000000000004B | 16777216 | 2024-05-06 12:37:51+05
 000000010000000000000049 | 16777216 | 2024-05-06 12:39:18+05
 00000001000000000000004A | 16777216 | 2024-05-06 12:38:08+05
(5 строк)
```
Сервер хранит журнальные файлы необходимые для восстановления 
не превышающие по объему минимальной отметки.
Т.к. в конфиге min_wal_size = 80MB - соответственно 5 файлов

## Попробуем нагрузочное тестирование в синхронном и асинхронном режиме
Сначала по умолчанию в синхронном режиме
```
oot@b1-t-oan:/etc/postgresql/15/main# sudo -u postgres pgbench -i buffer_temp
dropping old tables...
creating tables...
generating data (client-side)...
100000 of 100000 tuples (100%) done (elapsed 0.18 s, remaining 0.00 s)
vacuuming...
creating primary keys...
done in 0.48 s (drop tables 0.04 s, create tables 0.01 s, client-side generate 0.23 s, vacuum 0.09 s, primary keys 0.11 s).
root@b1-t-oan:/etc/postgresql/15/main# pgbench -P 1 -T 10 buffer_temp
pgbench: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: ВАЖНО:  роль "root" не существует
pgbench: error: could not create connection for setup
root@b1-t-oan:/etc/postgresql/15/main# sudo -u postgres pgbench -P 1 -T 10 -U postgres buffer_temp
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 1.0 s, 365.0 tps, lat 2.707 ms stddev 2.099, 0 failed
progress: 2.0 s, 563.0 tps, lat 1.777 ms stddev 0.637, 0 failed
progress: 3.0 s, 564.0 tps, lat 1.772 ms stddev 1.255, 0 failed
progress: 4.0 s, 510.0 tps, lat 1.959 ms stddev 1.005, 0 failed
progress: 5.0 s, 585.0 tps, lat 1.708 ms stddev 0.830, 0 failed
progress: 6.0 s, 579.0 tps, lat 1.727 ms stddev 0.835, 0 failed
progress: 7.0 s, 552.1 tps, lat 1.810 ms stddev 1.338, 0 failed
progress: 8.0 s, 569.0 tps, lat 1.758 ms stddev 1.182, 0 failed
progress: 9.0 s, 599.0 tps, lat 1.668 ms stddev 0.524, 0 failed
progress: 10.0 s, 477.0 tps, lat 2.054 ms stddev 2.372, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 5364
number of failed transactions: 0 (0.000%)
latency average = 1.863 ms
latency stddev = 1.348 ms
initial connection time = 10.788 ms
tps = 536.506926 (without initial connection time)

```
Затем в асинхронном режиме, предварительно перезапустив кластер

```
buffer_temp=# ALTER SYSTEM SET synchronous_commit = off;
ALTER SYSTEM
buffer_temp=# \q
root@b1-t-oan:/etc/postgresql/15/main# sudo -u postgres pgbench -P 1 -T 10 -U postgres buffer_temp
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 1.0 s, 542.0 tps, lat 1.830 ms stddev 0.607, 0 failed
progress: 2.0 s, 535.0 tps, lat 1.869 ms stddev 0.808, 0 failed
progress: 3.0 s, 557.0 tps, lat 1.793 ms stddev 0.815, 0 failed
progress: 4.0 s, 567.0 tps, lat 1.764 ms stddev 0.651, 0 failed
progress: 5.0 s, 528.0 tps, lat 1.894 ms stddev 1.938, 0 failed
progress: 6.0 s, 447.0 tps, lat 2.235 ms stddev 2.305, 0 failed
progress: 7.0 s, 567.0 tps, lat 1.763 ms stddev 0.574, 0 failed
progress: 8.0 s, 605.0 tps, lat 1.652 ms stddev 0.472, 0 failed
progress: 9.0 s, 542.0 tps, lat 1.843 ms stddev 0.967, 0 failed
progress: 10.0 s, 534.0 tps, lat 1.874 ms stddev 1.400, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 5425
number of failed transactions: 0 (0.000%)
latency average = 1.842 ms
latency stddev = 1.175 ms
initial connection time = 6.541 ms
tps = 542.756995 (without initial connection time)
root@b1-t-oan:/etc/postgresql/15/main# sudo pg_ctlcluster 15 main restart
root@b1-t-oan:/etc/postgresql/15/main# sudo -u postgres pgbench -P 1 -T 10 -U postgres buffer_temp
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 1.0 s, 830.8 tps, lat 1.194 ms stddev 0.294, 0 failed
progress: 2.0 s, 800.1 tps, lat 1.250 ms stddev 0.314, 0 failed
progress: 3.0 s, 770.0 tps, lat 1.298 ms stddev 0.706, 0 failed
progress: 4.0 s, 755.0 tps, lat 1.323 ms stddev 0.709, 0 failed
progress: 5.0 s, 658.0 tps, lat 1.519 ms stddev 0.910, 0 failed
progress: 6.0 s, 731.0 tps, lat 1.367 ms stddev 0.766, 0 failed
progress: 7.0 s, 773.0 tps, lat 1.293 ms stddev 0.528, 0 failed
progress: 8.0 s, 760.0 tps, lat 1.316 ms stddev 0.962, 0 failed
progress: 9.0 s, 813.1 tps, lat 1.229 ms stddev 0.273, 0 failed
progress: 10.0 s, 800.0 tps, lat 1.248 ms stddev 0.689, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 1
number of threads: 1
maximum number of tries: 1
duration: 10 s
number of transactions actually processed: 7692
number of failed transactions: 0 (0.000%)
latency average = 1.298 ms
latency stddev = 0.655 ms
initial connection time = 6.254 ms
tps = 769.666110 (without initial connection time)
root@b1-t-oan:/etc/postgresql/15/main# 

```
Наблюдаем увеличение TPS

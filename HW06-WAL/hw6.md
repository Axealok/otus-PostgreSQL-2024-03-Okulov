# Работа с журналами
Настраиваем создание контрольной точки каждые 30с
[Checkpoint timeout = 30s]()
Перезапускаем кластер
```
sudo pg_ctlcluster 15 main restart

```
Запускаем нагрузочный тест
```
root@b1-t-oan:/etc/postgresql/15/main# sudo -u postgres pgbench -c8 -P 30 -T 600 -U postgres postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg22.04+1))
starting vacuum...end.
progress: 30.0 s, 1028.8 tps, lat 7.744 ms stddev 5.620, 0 failed
progress: 60.0 s, 1062.3 tps, lat 7.514 ms stddev 5.214, 0 failed
progress: 90.0 s, 1001.4 tps, lat 7.971 ms stddev 6.044, 0 failed
progress: 120.0 s, 1016.4 tps, lat 7.852 ms stddev 5.648, 0 failed
progress: 150.0 s, 1004.0 tps, lat 7.953 ms stddev 5.762, 0 failed
progress: 180.0 s, 1069.6 tps, lat 7.462 ms stddev 5.177, 0 failed
progress: 210.0 s, 997.0 tps, lat 8.006 ms stddev 5.905, 0 failed
progress: 240.0 s, 1084.4 tps, lat 7.359 ms stddev 5.083, 0 failed
progress: 270.0 s, 1051.1 tps, lat 7.594 ms stddev 5.342, 0 failed
progress: 300.0 s, 1087.3 tps, lat 7.341 ms stddev 5.226, 0 failed
progress: 330.0 s, 1119.5 tps, lat 7.129 ms stddev 4.893, 0 failed
progress: 360.0 s, 1071.0 tps, lat 7.453 ms stddev 5.130, 0 failed
progress: 390.0 s, 985.0 tps, lat 8.103 ms stddev 5.941, 0 failed
progress: 420.0 s, 1048.4 tps, lat 7.610 ms stddev 5.229, 0 failed
progress: 450.0 s, 1057.8 tps, lat 7.547 ms stddev 5.191, 0 failed
progress: 480.0 s, 983.1 tps, lat 8.119 ms stddev 5.821, 0 failed
progress: 510.0 s, 1065.1 tps, lat 7.493 ms stddev 5.225, 0 failed
progress: 540.0 s, 1079.1 tps, lat 7.397 ms stddev 5.029, 0 failed
progress: 570.0 s, 1134.9 tps, lat 7.034 ms stddev 4.985, 0 failed
progress: 600.0 s, 1136.0 tps, lat 7.025 ms stddev 4.777, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 632481
number of failed transactions: 0 (0.000%)
latency average = 7.571 ms
latency stddev = 5.370 ms
initial connection time = 51.703 ms
tps = 1054.176752 (without initial connection time)

```
Смотрим информацию о кластере
```
  root@b1-t-oan:/etc/postgresql/15/main# sudo /usr/lib/postgresql/15/bin/pg_controldata /var/lib/postgresql/15/main/
Номер версии pg_control:              1300
Номер версии каталога:                202209061
Идентификатор системы баз данных:     7365743920985068351
Состояние кластера БД:                в работе
Последнее обновление pg_control:      Пн 06 мая 2024 11:24:16
Положение последней конт. точки:      0/2CE43D20
Положение REDO последней конт. точки: 0/2CE43CE8
Файл WAL c REDO последней к. т.:      00000001000000000000002C
Линия времени последней конт. точки:  1
Пред. линия времени последней к. т.:  1
Режим full_page_writes последней к.т: вкл.
NextXID последней конт. точки:        0:731317
NextOID последней конт. точки:        24647
NextMultiXactId послед. конт. точки:  1
NextMultiOffset послед. конт. точки:  0
oldestXID последней конт. точки:      716
БД с oldestXID последней конт. точки: 1
oldestActiveXID последней к. т.:      731317
oldestMultiXid последней конт. точки: 1
БД с oldestMulti последней к. т.:     1
oldestCommitTsXid последней к. т.:    0
newestCommitTsXid последней к. т.:    0
Время последней контрольной точки:    Пн 06 мая 2024 11:23:49
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

[Checkpoints]()


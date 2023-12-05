# PostgreSQL
Домашка 8

Нагрузочное тестирование и тюнинг PostgreSQL

Цель: сделать нагрузочное тестирование PostgreSQL

настроить параметры PostgreSQL для достижения максимальной производительности


Описание/Пошаговая инструкция выполнения домашнего задания:

развернуть виртуальную машину любым удобным способом - ***YC, 8 CPU, 32Gb RAM, SSD***

поставить на неё PostgreSQL 15 любым способом

настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины

нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)

написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему - ***лучший зафиксированный результат 1273 tps***

***Настройки***

max_connections = 100

shared_buffers = 8GB

effective_cache_size = 24GB

checkpoint_completion_target = 0.9

wal_buffers = 16MB

default_statistics_target = 100

random_page_cost = 1.1

effective_io_concurrency = 200

work_mem = 32Mb

min_wal_size = 2GB

max_wal_size = 8GB

maintenance_work_mem = 2GB

max_worker_processes = 8

max_parallel_workers_per_gather = 4

max_parallel_workers = 8

max_parallel_maintenance_workers = 4


Задание со *: аналогично протестировать через утилиту https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench)

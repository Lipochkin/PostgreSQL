# PostgreSQL
Домашнее задание
Работа с индексами

Цель:
знать и уметь применять основные виды индексов PostgreSQL
строить и анализировать план выполнения запроса
уметь оптимизировать запросы для с использованием индексов

Описание/Пошаговая инструкция выполнения домашнего задания:
Создать индексы на БД, которые ускорят доступ к данным.
В данном задании тренируются навыки:

определения узких мест
написания запросов для создания индекса
оптимизации
Необходимо:
Создать индекс к какой-либо из таблиц вашей БД
Прислать текстом результат команды explain,
в которой используется данный индекс
Реализовать индекс для полнотекстового поиска
Реализовать индекс на часть таблицы или индекс
на поле с функцией
Создать индекс на несколько полей
Написать комментарии к каждому из индексов
Описать что и как делали и с какими проблемами
столкнулись

Критерии оценки:
Критерии оценивания:
Выполнение ДЗ: 10 баллов
плюс 2 балла за красивое решение
минус 2 балла за рабочее решение, и недостатки указанные преподавателем не устранены


Использовал БД:


![2](https://github.com/Lipochkin/PostgreSQL/assets/147322715/6d8f247e-0577-40cb-a5e4-166e48e7d7a5)

Создано три индекса:

Создано три индекса:
- Использовал запросы select по индексируемым полям
- Индекс для полнотекстового поиска в сущности Product по полю name. __idx_product_name__
- create index idx_product_name on product(name)

__До индекса__
```
 Seq Scan on product  (cost=0.00..2310.40 rows=100022 width=33) (actual time=0.013..55.433 rows=100003 loops=1)
   Filter: ((name)::text ~~* '%v.11.0.%'::text)
   Rows Removed by Filter: 29
 Planning Time: 0.187 ms
 Execution Time: 57.629 ms
```

__После индекса__
```
 Seq Scan on product  (cost=0.00..2310.40 rows=100022 width=33) (actual time=0.013..45.889 rows=100003 loops=1)
   Filter: ((name)::text ~~* '%v.11.0.%'::text)
   Rows Removed by Filter: 29
 Planning Time: 0.171 ms
 Execution Time: 47.881 ms
```

Время исполнения запроса быстрее  на 10 сек.


- Индекс на часть таблицы Shop_Point_Products по полю count_products __idx_shop_point_products_count_products__(индекс btree)
- create index idx_shop_point_products_count_products on Shop_Point_Products(count_products)

__До индекса__
``` 
 Hash Join  (cost=4093.86..20432.07 rows=320062 width=55) (actual time=20.619..156.567 rows=319633 loops=1)
   Hash Cond: (spp.fk_shop_point = sp.id)
   ->  Hash Join  (cost=4092.72..18997.32 rows=320062 width=37) (actual time=20.607..123.314 rows=319633 loops=1)
         Hash Cond: (spp.fk_product = p.id)
         ->  Seq Scan on shop_point_product spp  (cost=0.00..10780.40 rows=320062 width=8) (actual time=0.003..42.368
 rows=319633 loops=1)
               Filter: (count_products < '18'::double precision)
               Rows Removed by Filter: 280559
         ->  Hash  (cost=2060.32..2060.32 rows=100032 width=37) (actual time=20.338..20.339 rows=100032 loops=1)
               Buckets: 131072  Batches: 2  Memory Usage: 4537kB
               ->  Seq Scan on product p  (cost=0.00..2060.32 rows=100032 width=37) (actual time=0.002..7.222 rows=10
0032 loops=1)
   ->  Hash  (cost=1.06..1.06 rows=6 width=26) (actual time=0.009..0.009 rows=6 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         ->  Seq Scan on shop_point sp  (cost=0.00..1.06 rows=6 width=26) (actual time=0.003..0.004 rows=6 loops=1)
 Planning Time: 0.326 ms
 Execution Time: 162.694 ms
```

__После индекса__
```
 Hash Join  (cost=4093.86..20432.07 rows=320062 width=55) (actual time=19.552..151.628 rows=319633 loops=1)
   Hash Cond: (spp.fk_shop_point = sp.id)
   ->  Hash Join  (cost=4092.72..18997.32 rows=320062 width=37) (actual time=19.541..117.358 rows=319633 loops=1)
         Hash Cond: (spp.fk_product = p.id)
         ->  Seq Scan on shop_point_product spp  (cost=0.00..10780.40 rows=320062 width=8) (actual time=0.003..39.048
 rows=319633 loops=1)
               Filter: (count_products < '18'::double precision)
               Rows Removed by Filter: 280559
         ->  Hash  (cost=2060.32..2060.32 rows=100032 width=37) (actual time=19.477..19.478 rows=100032 loops=1)
               Buckets: 131072  Batches: 2  Memory Usage: 4537kB
               ->  Seq Scan on product p  (cost=0.00..2060.32 rows=100032 width=37) (actual time=0.002..6.740 rows=10
0032 loops=1)
   ->  Hash  (cost=1.06..1.06 rows=6 width=26) (actual time=0.008..0.008 rows=6 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         ->  Seq Scan on shop_point sp  (cost=0.00..1.06 rows=6 width=26) (actual time=0.004..0.005 rows=6 loops=1)
 Planning Time: 0.299 ms
 Execution Time: 157.518 ms
```

Время исполнения запроса быстрее на 5 сек.


- Индекс на часть таблицы Work_Shift по полям fk_staff, fk_shop_point (индекс btree)
- create index idx_ord_order_date_inc_status on Work_Shift(fk_staff) include (fk_shop_point)
  
__До индекса__
``` 
 Hash Join  (cost=3.49..6.22 rows=100 width=33) (actual time=0.048..0.079 rows=100 loops=1)
   Hash Cond: (ws.fk_shop_point = sp.id)
   ->  Hash Join  (cost=2.35..4.63 rows=100 width=15) (actual time=0.030..0.049 rows=100 loops=1)
         Hash Cond: (ws.fk_staff = st.id)
         ->  Seq Scan on work_shift ws  (cost=0.00..2.00 rows=100 width=12) (actual time=0.006..0.010 rows=100 loops=
1)
         ->  Hash  (cost=1.60..1.60 rows=60 width=11) (actual time=0.016..0.016 rows=90 loops=1)
               Buckets: 1024  Batches: 1  Memory Usage: 12kB
               ->  Seq Scan on staff st  (cost=0.00..1.60 rows=60 width=11) (actual time=0.002..0.008 rows=90 loops=1
)
   ->  Hash  (cost=1.06..1.06 rows=6 width=26) (actual time=0.008..0.008 rows=6 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         ->  Seq Scan on shop_point sp  (cost=0.00..1.06 rows=6 width=26) (actual time=0.004..0.004 rows=6 loops=1)
 Planning Time: 0.479 ms
 Execution Time: 0.109 ms
```

__После индекса__
```
 Hash Join  (cost=3.49..6.22 rows=100 width=33) (actual time=0.030..0.060 rows=100 loops=1)
   Hash Cond: (ws.fk_shop_point = sp.id)
   ->  Hash Join  (cost=2.35..4.63 rows=100 width=15) (actual time=0.019..0.037 rows=100 loops=1)
         Hash Cond: (ws.fk_staff = st.id)
         ->  Seq Scan on work_shift ws  (cost=0.00..2.00 rows=100 width=12) (actual time=0.002..0.006 rows=100 loops=
1)
         ->  Hash  (cost=1.60..1.60 rows=60 width=11) (actual time=0.014..0.014 rows=90 loops=1)
               Buckets: 1024  Batches: 1  Memory Usage: 12kB
               ->  Seq Scan on staff st  (cost=0.00..1.60 rows=60 width=11) (actual time=0.002..0.007 rows=90 loops=1
)
   ->  Hash  (cost=1.06..1.06 rows=6 width=26) (actual time=0.007..0.007 rows=6 loops=1)
         Buckets: 1024  Batches: 1  Memory Usage: 9kB
         ->  Seq Scan on shop_point sp  (cost=0.00..1.06 rows=6 width=26) (actual time=0.003..0.004 rows=6 loops=1)
 Planning Time: 0.227 ms
 Execution Time: 0.077 ms
```

Время исполнения запроса быстрее на 30 мили. сек.

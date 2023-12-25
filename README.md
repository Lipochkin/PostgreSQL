# PostgreSQL
Домашка 10-11

Домашнее задание 10
Репликация

Цель: реализовать свой миникластер на 3 ВМ.


Описание/Пошаговая инструкция выполнения домашнего задания:

На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение. ***-ок***

Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2. ***-при попытке подписаться была ошибка, создал отдельного юзера для подписки и все сработало***

На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение. ***-ок***

Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1. ***-ок***

3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ). ***-ок***

ДЗ сдается в виде миниотчета на гитхабе с описанием шагов и с какими проблемами столкнулись. ***-больше проблем не было***

реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись. ***-не стал делать, сделал еще одну домашку, чтобы до НГ успеть =(***


***------***


Домашнее задание 11
Работа с join'ами, статистикой

Цель:

знать и уметь применять различные виды join'ов

строить и анализировать план выполнения запроса

оптимизировать запрос

уметь собирать и анализировать статистику для таблицы

Описание/Пошаговая инструкция выполнения домашнего задания:

В результате выполнения ДЗ вы научитесь пользоваться различными вариантами соединения таблиц.

В данном задании тренируются навыки: написания запросов с различными типами соединений

Необходимо:

Реализовать прямое соединение двух или более таблиц:

***Города ТОП3***
```
select a.aircraft_code, dep.city departure, arr.city arrive, count(*)
from bookings.aircrafts a
         join bookings.flights f on a.aircraft_code = f.aircraft_code
         join bookings.airports dep on dep.airport_code = f.departure_airport
         join bookings.airports arr on arr.airport_code = f.departure_airport
limit 3
```

Реализовать левостороннее (или правостороннее) соединение двух или более таблиц, реализовать запрос, в котором будут использованы
разные типы соединений:

***Максимально заполненные самолеты ТОП3***
```
select f.flight_id, count(tf.*) / count(s.*) * 100 from bookings.flights f
join bookings.aircrafts a on f.aircraft_code = a.aircraft_code
join bookings.seats s on a.aircraft_code = s.aircraft_code
left join bookings.ticket_flights tf on f.flight_id = tf.flight_id
limit 3
```

Реализовать кросс соединение двух или более таблиц:

***Максимально удаленные друг от друга аэропорты***
```
select a.city, b.city, (point(a.longitude,a.latitude) <@> point(b.longitude,b.latitude)) as distance from bookings.airports a
cross join bookings.airports b
order by distance desc
limit 1
```
Реализовать полное соединение двух или более таблиц

***Просмотр всех мест в каждом самолете***
```
select * from aircrafts b
full join seats mb on aircraft_code=mb.id
order by aircraft_code desc
```

Сделать комментарии на каждый запрос

К работе приложить структуру таблиц, для которых выполнялись соединения 

***Взял готовую БД - https://edu.postgrespro.ru/demo_small.zip***

![schema](https://postgrespro.ru/media/docs/postgrespro/9.6/ru/demodb-bookings-schema.svg)

Задание со звездочкой*
Придумайте 3 своих метрики на основе показанных представлений, отправьте их через ЛК, а так же поделитесь с коллегами в слаке

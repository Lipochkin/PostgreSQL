# PostgreSQL
Домашка 9

Домашнее задание

Бэкапы

Цель: Применить логический бэкап. Восстановиться из бэкапа


Описание/Пошаговая инструкция выполнения домашнего задания:

Создаем ВМ/докер c ПГ. ***- ок***

Создаем БД, схему и в ней таблицу. ***- ок***

Заполним таблицы автосгенерированными 100 записями. ***- ок***

Под линукс пользователем Postgres создадим каталог для бэкапов ***- ок***

Сделаем логический бэкап используя утилиту COPY ***- ок***

Восстановим в 2 таблицу данные из бэкапа. ***- ок***

Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц ***- ок***

Используя утилиту pg_restore восстановим в новую БД только вторую таблицу! ***-не сработало, даже при повторном pg_dump cо сменой названия схемы и таблиц, сработало только после ручного создания схемы в другой БД***

ДЗ сдается в виде миниотчета на гитхабе с описанием шагов и с какими проблемами столкнулись.

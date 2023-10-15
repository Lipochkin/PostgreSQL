# PostgreSQL
Домашка
Установка и настройка PostgteSQL в контейнере Docker

Цель:
установить PostgreSQL в Docker контейнере
настроить контейнер для внешнего подключения

Описание/Пошаговая инструкция выполнения домашнего задания:
создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом

поставить на нем Docker Engine

сделать каталог /var/lib/postgres

развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql

развернуть контейнер с клиентом postgres

подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк

подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP/ЯО/места установки докера

удалить контейнер с сервером

создать его заново

подключится снова из контейнера с клиентом к контейнеру с сервером

проверить, что данные остались на месте

оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами - **не было проблем)**

![Скрин](https://github.com/Lipochkin/PostgreSQL/blob/main/1.PNG)

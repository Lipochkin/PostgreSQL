# PostgreSQL
Домашка

Домашнее задание 6 


Работа с журналами

Цель:

уметь работать с журналами и контрольными точками

уметь настраивать параметры журналов


Описание/Пошаговая инструкция выполнения домашнего задания:

Настройте выполнение контрольной точки раз в 30 секунд.

10 минут c помощью утилиты pgbench подавайте нагрузку.

Измерьте, какой объем журнальных файлов был сгенерирован за это время. - ***123 Mb***

Оцените, какой объем приходится в среднем на одну контрольную точку. - ***~8 Mb***

Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло? - ***не все, раза в 4 реже выполнялись, возможно из-за нагрузки не успевали***

Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат. - ***раза в три выше tps, реже и организованней идет запись изменений во втором случае***

Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. 

Включите кластер и сделайте выборку из таблицы. Что и почему произошло? - ***проверка целостности включена, поэтому postgres ругается на повреждения***

как проигнорировать ошибку и продолжить работу? - ***удалить поврежденные записи вручную либо проигнорировать и продолжить работу, отключив проверку ignore_checksum_failure=on***

_________________________________________________


Домашнее задание 7

Механизм блокировок

Цель: понимать как работает механизм блокировок объектов и строк


Описание/Пошаговая инструкция выполнения домашнего задания:

Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд. 

Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.

***postgres@DESKTOP-3FFOGTK:/home/fkdark$ cat /var/log/postgresql/postgresql-15-test.log | grep "deadlock detected" -A 10
2023-12-03 19:00:44.563 MSK [1779] postgres@postgres ERROR:  deadlock detected
2023-12-03 19:00:44.563 MSK [1779] postgres@postgres DETAIL:  Process 1779 waits for ShareLock on transaction 748; blocked by process 1774.
        Process 1774 waits for ShareLock on transaction 749; blocked by process 1779.
        Process 1779: UPDATE test SET message = 'message from session 222' WHERE id = 1;
        Process 1774: UPDATE test SET message = 'session 111' WHERE id = 2;
2023-12-03 19:00:44.563 MSK [1779] postgres@postgres HINT:  See server log for query details.
2023-12-03 19:00:44.563 MSK [1779] postgres@postgres CONTEXT:  while updating tuple (0,7) in relation "test"
2023-12-03 19:00:44.563 MSK [1779] postgres@postgres STATEMENT:  UPDATE test SET message = 'message from session 222' WHERE id = 1;***

Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сессиях. 

Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны. 

Пришлите список блокировок и объясните, что значит каждая.

***сессия pid	
1	   1774	
2	   1779	
3	   1783***

 relation      |        5 |    16400 |      |       |            |               |         |       |          | 6/7                | 1783 | RowExclusiveLock | t       | t        |

 relation      |        5 |    16395 |      |       |            |               |         |       |          | 6/7                | 1783 | RowExclusiveLock | t       | t        |

 virtualxid    |          |          |      |       | 6/7        |               |         |       |          | 6/7                | 1783 | ExclusiveLock    | t       | t        |

 relation      |        5 |    16400 |      |       |            |               |         |       |          | 5/12               | 1779 | RowExclusiveLock | t       | t        |

 relation      |        5 |    16395 |      |       |            |               |         |       |          | 5/12               | 1779 | RowExclusiveLock | t       | t        |

 virtualxid    |          |          |      |       | 5/12       |               |         |       |          | 5/12               | 1779 | ExclusiveLock    | t       | t        |

 relation      |        5 |    16400 |      |       |            |               |         |       |          | 4/23               | 1774 | RowExclusiveLock | t       | t        |

 relation      |        5 |    16395 |      |       |            |               |         |       |          | 4/23               | 1774 | RowExclusiveLock | t       | t        |

 virtualxid    |          |          |      |       | 4/23       |               |         |       |          | 4/23               | 1774 | ExclusiveLock    | t       | t        |

 transactionid |          |          |      |       |            |           759 |         |       |          | 4/23               | 1774 | ExclusiveLock    | t       | f        |

 tuple         |        5 |    16395 |    0 |    11 |            |               |         |       |          | 6/7                | 1783 | ExclusiveLock    | f       | f        | 2023-12-03 19:57:40.494189+03
 
 transactionid |          |          |      |       |            |           760 |         |       |          | 5/12               | 1779 | ExclusiveLock    | t       | f        |

 transactionid |          |          |      |       |            |           761 |         |       |          | 6/7                | 1783 | ExclusiveLock    | t       | f        |

 tuple         |        5 |    16395 |    0 |    11 |            |               |         |       |          | 5/12               | 1779 | ExclusiveLock    | t       | f        |

 transactionid |          |          |      |       |            |           759 |         |       |          | 5/12               | 1779 | ShareLock        | f       | f        |

***Каждая сессия захватила эксклюзивную блокировку строки и ключа, первая (pid 1774) блокирует остальные
Каждая сессия держит exclusive lock номеров своих транзакций и виртуальной транзакции virtualxid
Вторая и третья сессии повесили ExclusiveLock на tuple, из-за того, что он был обновлен первой сессией и они ждут в очереди.
ShareLock в конце из-за того, что мы пытаемся обновить ту же строку что и в первой сессии, когда очередь дойдет до второй - третья повесит такую же блокировку.***

Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений? - ***нет, можно только увидеть что блокировалось и что пытались обновить***

Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга? - ***да, если поставить repeatable read и они будут обновлять значения в одинаковой строке, например с условием select for update из той же таблицы***

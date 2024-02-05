**<div align="center"><h2>Морозова Юлия, OTUS, группа Postgre-DBA-2023-11</h2></div>**

**<div align=center><h3>Домашнее задание №5 по теме:</h3></div>**
**<div align=center><h3>«MVCC, vacuum и autovacuum.»</h3></div>**

***
**<h3>Домашнее задание:
<br>Настройка autovacuum с учетом особеностей производительности</h3>**

**<h3>Цель:
<br>запустить нагрузочный тест pgbench
<br>настроить параметры autovacuum
<br>проверить работу autovacuum.</h3>**

***

**Выполнение:**

>**1. Создать инстанс ВМ с 2 ядрами и 4 Гб ОЗУ и SSD 10GB**

В ЯО создала новую ВМ для этого домашнего задания, но размер диска не сделала 30 ГБ(ЯО не дал возможности создать меньше, думаю, что это не критично):

  ![1_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/46d076d6-1940-41bf-a039-38c5cef40f9d)

<br/>

>**2. Установить на него PostgreSQL 15 с дефолтными настройками**

Подключаюсь к этой ВМ в ЯО по ``ssh`` с помощью putty(ключи public , private генерирую с помощью puttygen) и устанавливаю Postgres 15й версии с настройками по умолчанию командами и далее проверяю:

``sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15``

  ![2_2](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/eb835560-0e8b-49ae-9237-e9b4a8263c54)

<br/>

>**3. Создать БД для тестов: выполнить pgbench -i postgres**

выполняю: ``sudo -u postgres pgbench -i postgres``

  ![3_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/afa29395-577d-4171-9087-6ea90a068b2e)

  <br/>

>**4. Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres**

выполняю: ``sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres``

  ![4_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/0e8c2707-f39f-4a4d-be50-0b9845c94bf8)

  <br/>

>**5. Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла**


Сначала смотрю текущие значения параметров, какие буду менять далее, смотрю запросом:

```sql
    select name, setting, context from pg_settings where name in(
      'max_connections',
      'shared_buffers',
      'effective_cache_size',
      'maintenance_work_mem',
      'checkpoint_completion_target',
      'wal_buffers',
      'default_statistics_target',
      'random_page_cost',
      'effective_io_concurrency',
      'work_mem',
      'min_wal_size',
      'max_wal_size');
```

сейчас они по умолчанию:  
  ![5_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/a78cbf99-1057-447d-97c3-da2b8400bdda)

выставляю их (согласно заданию) скриптом:

  ```sql
    ALTER SYSTEM SET
    max_connections='40';
    ALTER SYSTEM SET
    shared_buffers='1GB';
    ALTER SYSTEM SET
    effective_cache_size='3GB';
    ALTER SYSTEM SET
    maintenance_work_mem='512MB';
    ALTER SYSTEM SET
    checkpoint_completion_target='0.9';
    ALTER SYSTEM SET
    wal_buffers='16MB';
    ALTER SYSTEM SET
    default_statistics_target='500';
    ALTER SYSTEM SET
    random_page_cost='4';
    ALTER SYSTEM SET
    effective_io_concurrency='2';
    ALTER SYSTEM SET
    work_mem='6553kB';
    ALTER SYSTEM SET
    min_wal_size='4GB';
    ALTER SYSTEM SET
    max_wal_size='16GB';
  ```

  ![5_2](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/5e26386e-585b-4e99-aa5b-5e0c95778cbf)

Так как среди измененных есть параметры, требующие рестарта, то сначала выполняю команду: 
<br> ``sudo -u postgres pg_ctlcluster 15 main restart``, потом захожу в postgres и проверяю параметры скриптом:

```sql
    select name, setting, unit, context from pg_settings where name in(
      'max_connections',
      'shared_buffers',
      'effective_cache_size',
      'maintenance_work_mem',
      'checkpoint_completion_target',
      'wal_buffers',
      'default_statistics_target',
      'random_page_cost',
      'effective_io_concurrency',
      'work_mem',
      'min_wal_size',
      'max_wal_size');
```

и все ок:

![5_4](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/0a4e866a-df0d-4751-87aa-8780dfe42ed0)

<br/>
  
>**6. Протестировать заново**

выполняю: ``sudo -u postgres pgbench -c8 -P 6 -T 60 -U postgres postgres``

  ![image](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/75d03063-a4ea-4709-b9ad-1ab77d5126cc)

<br/> 

>**7. Что изменилось и почему?**

Наблюдаю значительный прирост производительности: увеличение порядка 25-26% показателя tps(показатель пропускной способности БД - показывает количество транзакций, обработанных базой за одну секунду) и так же вижу снижение latency почти на 20% (время выполнения для каждого оператора).
<br>Этот прирост объясняю тем, что postgres настроен теперь более оптимально и согласно текущей конфигурации оборудования (увеличен буффер разделяемой памяти, доступной серверу, увеличен дисковый кеш запроса, увеличен maintenance_work_mem, увеличено допустимое число параллельных операций ввода/вывода). 

То есть новые настройки позволят нам использовать оба ядра, больше памяти для сортировки, больше дискового кеша для запросов, возможность эффективной параллельной работы.

<br/> 

>**8. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк**

выполняю командой:
```sql
  CREATE TABLE test_autovacuum(txt_data text);
```

  ![8_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/5e2a9a31-6a21-4dcc-a773-ca782d504df4)

<br/>

>**9. Посмотреть размер файла с таблицей**

выполняю командой:
```sql
  select pg_size_pretty(pg_total_relation_size('test_autovacuum'));
```

  ![9_1](https://github.com/Y-M-Morozova/8_homework_Morozova_Yulia/assets/153178571/7581b514-c3b8-4abb-8947-0d90f5b5ba6b)

<br/>

>**10. 5 раз обновить все строчки и добавить к каждой строчке любой символ**



<br/>

>**11. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум**


<br/>

>**12. Подождать некоторое время, проверяя, пришел ли автовакуум**


<br/>

>**13. 5 раз обновить все строчки и добавить к каждой строчке любой символ**


<br/>

>**14. Посмотреть размер файла с таблицей**


<br/>

>**15. Отключить Автовакуум на конкретной таблице**


<br/>


>**16. 10 раз обновить все строчки и добавить к каждой строчке любой символ**



<br/>

>**17. Посмотреть размер файла с таблицей**



<br/>

>**18. Объясните полученный результат**



<br/>

>**19. Не забудьте включить автовакуум)**



<br/>  


***
**<h3> Задание со * :
<br>Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
<br>Не забыть вывести номер шага цикла</h3>**

***






>**20. подсказка в шпаргалке под пунктом 20**

О, да! Кратко и понятно - посмотрела в шпаргалке, да: 

***таблица создана в схеме public а не testnm и прав на public для роли readonly не давали***

<br/> 

>**21. а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)**

 Потому,  что я осознанно сделала выбор - создавать таблицу в схеме по умолчанию, а это ``public``, не указывала схему  при создании таблицы как ``testnm``

<br/>

>**22. вернитесь в базу данных testdb под пользователем postgres**

  ![22_1](https://github.com/Y-M-Morozova/4_homework_Morozova_Yulia/assets/153178571/eb9c029e-75b2-4ccb-b941-eba30ecab63e)

<br/>

>**23. удалите таблицу t1**

выполняю командой и потом проверяю:

```sql
  drop table t1;
```

  ![23_1](https://github.com/Y-M-Morozova/4_homework_Morozova_Yulia/assets/153178571/e671cb69-10ae-4acd-b389-aad34e8053d6)

<br/>

>**24. создайте ее заново но уже с явным указанием имени схемы testnm**

выполняю командой и потом проверяю:

```sql
  create table testnm.t1 (c1 integer);
```

  ![24_1](https://github.com/Y-M-Morozova/4_homework_Morozova_Yulia/assets/153178571/d72c0d57-fc0a-4baa-a97b-0e2d0d421b1d)

<br/>

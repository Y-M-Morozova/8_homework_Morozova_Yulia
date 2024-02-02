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


<br/>

>**2. Установить на него PostgreSQL 15 с дефолтными настройками**

<br/>

>**3. Создать БД для тестов: выполнить pgbench -i postgres**


  <br/>

>**4. Запустить pgbench -c8 -P 6 -T 60 -U postgres postgres**



  <br/>

>**5. Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла**




<br/>
  
>**6. Протестировать заново**


 <br/> 

>**7. Что изменилось и почему?**


<br/> 

>**8. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк**


<br/>

>**9. Посмотреть размер файла с таблицей**


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

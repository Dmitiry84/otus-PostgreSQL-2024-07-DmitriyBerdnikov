# Домашнее задание 1.
# Работа с уровнями изоляции транзакции в PostgreSQL.

### 1. Подключаемся по SSH, окно терминала № 1:

```
ssh -i C:\\Users\\Diman\\bdv bdv@91.224.87.19

System information as of Fri Aug  9 17:26:47 UTC 2024

System load:  0.0               Processes:               142
Usage of /:   8.3% of 28.89GB   Users logged in:         0
Memory usage: 7%                IPv4 address for enp3s0: 10.0.0.5
Swap usage:   0%
```

### 2. Подключаемся по SSH, окно терминала № 2:

```
ssh -i C:\\Users\\Diman\\bdv bdv@91.224.87.19

System information as of Fri Aug  9 17:33:38 UTC 2024

System load:  0.06              Processes:               148
Usage of /:   8.3% of 28.89GB   Users logged in:         1
Memory usage: 7%                IPv4 address for enp3s0: 10.0.0.5
Swap usage:   0%
```

### 3. Подключаемся к БД postgres, окно терминала № 1:

```
sudo -u postgres psql

psql (16.4 (Ubuntu 16.4-1.pgdg22.04\+1))
Type "help" for help.

postgres=#
```

### 4. Подключаемся к БД postgres, окно терминала № 2:

```
sudo -u postgres psql

psql (16.4 (Ubuntu 16.4-1.pgdg22.04\+1))
Type "help" for help.
```

### 5. Отключаем автофиксацию на обоих терминалах с помощью команды:

```
\set AUTOCOMMIT false
```

### 6. Проверяем с помощью команды:

```
postgres=# \echo :AUTOCOMMIT
false
```

### 7. Создать таблицу **persons** и наполнить данными:

```
postgres=# create table persons(id serial, first_name text, second_name text);
CREATE TABLE
postgres=*#
```

### 8. Добавить строки в таблицу:

```
postgres=*# insert into persons(first_name, second_name) values('ivan', 'ivanov'),('petr', 'petrov');
INSERT 0 2
postgres=*#
```

### 9. Подтвердить транзакцию:

```
postgres=*# commit;
COMMIT
postgres=#
```

### 10. Посмотреть текущий уровень изоляции: show transaction isolation level:

```
postgres=# show transaction isolation level;

transaction_isolation

read committed
(1 row)
```

### 11. В первой сессии добавить новую запись:
### insert into persons(first_name, second_name) values('sergey', 'sergeev'):

```
postgres=*# insert into persons(first_name, second_name) values ('sergey','sergeev');
INSERT 0 1
postgres=*#
```

### 12. Смотрим таблицу во 2й сессии:

```
postgres=# select * from persons;

id | first_name | second_name
----+------------+-------------
1 | ivan       | ivanov   
2 | petr       | petrov
(2 rows)
```

**{green}(ОТВЕТ:)** Во второй сесси не видим новую запись, т.к. транзакция ещё нефиксирована и по умолчанию уровень изоляции транзакций установлен **read committed**.
Т.е. читаем только зафиксированные транзакции.

### 14. Завершаем транзакцию в 1й сессии:

```
postgres=*# commit;
COMMIT
postgres=#
```

### 15. Смотрим таблицу во 2й сессии:

```
postgres=*# select * from persons;
id | first_name | second_name
----+------------+-------------
1 | ivan       | ivanov
2 | petr       | petrov
3 | sergey     | sergeev
(3 rows)

postgres=*#
```

**{green}(ОТВЕТ:)** После подтверждения транзации в 1й сессии в режиме **read commited** мы можем видеть все зафиксированные транзации.

### 16. В двух сессиях начать новые, но уже repeatable read транзакции - set transaction isolation level repeatable read;

```
postgres=# begin transaction isolation level repeatable read;
BEGIN
postgres=*#
```
### 17. Добавляем запись в таблицу в 1й сессии:
```
postgres=*# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
postgres=*#
```
### 18. Смотрим таблицу на 2м терминале:

```
postgres=*# select * from persons;
id | first_name | second_name
----+------------+-------------
1 | ivan       | ivanov
2 | petr       | petrov
3 | sergey     | sergeev
(3 rows)

postgres=*#
```

**{green}(ОТВЕТ:)** Не видим запись, т.к. транзакция в 1й сессии ещё не зафиксирована. 
И будут отображаться данные, считанные на момент начала транзанции, т.к. уровень транзакции repeatable read.

### 19.Фиксируем транзакцию в 1й сессии:

```
postgres=*# commit;
COMMIT
postgres=#
```

### 20. Смотрим в таблицу на 2м терминале:

```
postgres=*# select * from persons;
id | first_name | second_name
----+------------+-------------
1 | ivan       | ivanov
2 | petr       | petrov
3 | sergey     | sergeev
(3 rows)

postgres=*#
```

**{green}(ОТВЕТ:)** Не видим зафиксированную транзакцию,
т.к. уровень изоляции открытой транзакции во 2й сессии repeatable read -
видит данные, которые были считаны до начала транзакции.

Фиксируем транзакцию во 2й сессии и снова смотрим в таблицу:
т.к. транзакция в 1й сессии зафиксирована и во 2й сессии мы закрыли транзацию,
то при вызове команды select происходит чтение всех зафиксированных транзакций.

```
postgres=*# select * from persons; commit;
id | first_name | second_name
----+------------+-------------
1 | ivan       | ivanov
2 | petr       | petrov
3 | sergey     | sergeev
4 | sveta      | svetova
(4 rows)

COMMIT
postgres=#
```

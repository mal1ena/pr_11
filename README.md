# pr_11 Производительный SQL
## Цель
Изучение методов оптимизации SQL-запросов для повышения производительности базы данных, включая анализ планов выполнения запросов с помощью EXPLAIN ANALYZE, применение различных типов индексов (B-tree, Hash, Bitmap), а также эффективное использование операций соединения (JOIN).

### 1.1 Используйте команду EXPLAIN, чтобы вернуть план запроса для выбора всех доступных записей в таблице клиентов.
```
EXPLAIN SELECT * FROM customers;
```
![image](https://github.com/user-attachments/assets/b8b4c937-5fb1-4bd6-8cd9-49d73f8793e0)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
  -начальные затраты составили 0.00.., сообщаемая стоимость составила 1540.00
  -общая стоимость выполнения запроса - 50000
  -общее количество строк, которые доступны для возврата - 140

### 1.2 Повторите запрос из шага 2 этого упражнения, на этот раз ограничив количество возвращаемых записей до 15.
```
EXPLAIN SELECT * FROM customers LIMIT 15;
```
![image](https://github.com/user-attachments/assets/1a5ba182-15a0-45ee-8256-38cac1ee9d39)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
  -начальные затраты составили 0.00.., сообщаемая стоимость составила 1540.00
  -общая стоимость выполнения запроса - 50000
  -общее количество строк, которые доступны для возврата - 140
Можно сделать вывод, что ограничение количества строк никак не повлияло на стоимость запроса

### 1.3 Создайте план запроса, выбрав все строки, где клиенты живут в пределах широты 30 и 40 градусов.
```
EXPLAIN SELECT * FROM customers WHERE latitude > 30 and latitude < 40;
```
![image](https://github.com/user-attachments/assets/0a3a7f91-84ac-476d-be3b-e20e632337df)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
   -начальные затраты составили 0.00.., сообщаемая стоимость составила 1790.00
   -общая стоимость выполнения запроса - 26291
   -общее количество строк, которые доступны для возврата - 140
Данный запрос требует больше вычислительных ресурсов и больше памяти для передачи данных

### 2.1 Используйте команды EXPLAIN и ANALYZE, чтобы профилировать план запроса для поиска всех записей с IP-адресом 18.131.58.65. Сколько времени занимает планирование и выполнение запроса?
```
EXPLAIN ANALYZE SELECT * FROM customers WHERE ip_address = '18.131.58.65'
```
![image](https://github.com/user-attachments/assets/82197581-1e34-47a8-b227-5dcd391608ea)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
   -начальные затраты составили 0.00.., сообщаемая стоимость составила 1665.00
   -общая стоимость выполнения запроса - 1
   -общее количество строк, которые доступны для возврата - 140
   -ожидаемое время выполнения запроса составило 0,153 мс, полученное время выполнения составило 13,839 мс

### 2.2 Создайте общий индекс на основе столбца IP-адреса.
```
CREATE INDEX ix_address ON customers(ip_address)
```
### 2.3 Повторите запрос шага 1. Сколько времени занимает планирование и выполнение запроса?
```
EXPLAIN ANALYZE SELECT * FROM customers WHERE ip_address = '18.131.58.65'
```
![image](https://github.com/user-attachments/assets/37017dfa-7e2c-4e45-9127-71c2bf28bba3)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
   -начальные затраты составили 0.00.., сообщаемая стоимость составила 1665.00
   -общая стоимость выполнения запроса - 1
   -общее количество строк, которые доступны для возврата - 140
   -ожидаемое время выполнения запроса составило 2,739 мс, полученное время выполнения составило 0,165 мс
С помощью индекса время выполнения запроса уменьшилось в 13 раз

### 2.4 Создайте более подробный индекс на основе столбца IP-адреса с условием, что IP-адрес равен 18.131.58.65.
```
CREATE INDEX ix_address_detal ON customers(ip_address) WHERE ip_address = '18.131.58.65'
```

### 2.5 Повторите запрос шага 1. Сколько времени занимает планирование и выполнение запроса? Каковы различия между каждым из этих запросов?
```
EXPLAIN ANALYZE SELECT * FROM customers WHERE ip_address = '18.131.58.65'
```
![image](https://github.com/user-attachments/assets/86061185-5e14-4b58-9f66-51415781a41b)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
   -начальные затраты составили 0.12.., сообщаемая стоимость составила 8,14
   -общая стоимость выполнения запроса - 1
   -общее количество строк, которые доступны для возврата - 140
   -ожидаемое время выполнения запроса составило 3,097 мс, полученное время выполнения составило 0,115 мс
С помощью индекса время выполнения запроса уменьшилось в 13 раз
Стоимость запроса и время его выполнения также уменьшились, но несильно.

### 2.6 Используйте команды EXPLAIN и ANALYZE для профилирования плана запроса для поиска всех записей с суффиксом Jr. Сколько времени занимает планирование и выполнение запроса?
```
EXPLAIN ANALYZE SELECT * FROM customers WHERE suffix = 'Jr'
```
![image](https://github.com/user-attachments/assets/50f7b9f6-05c3-4ff6-8c6a-a80dee2839db)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
   -начальные затраты составили 0.00.., сообщаемая стоимость составила 1665,00
   -общая стоимость выполнения запроса - 100
   -общее количество строк, которые доступны для возврата - 140
   -ожидаемое время выполнения запроса составило 0,315 мс, полученное время выполнения составило 12,811 мс

### 2.7 Создайте общий индекс на основе столбца адреса суффикса.
```
CREATE INDEX ix_suffix ON customers(suffix)
```

### 2.8 Повторно запустите запрос шага 6. Сколько времени занимает планирование и выполнение запроса?
```
EXPLAIN ANALYZE SELECT * FROM customers WHERE suffix = 'Jr'
```
![image](https://github.com/user-attachments/assets/c1025ac7-be5a-4493-91ff-f3c5247c71b9)

План запроса читается снизу вверх, так как представляется собой дерево.
Был произведен Seq Scan, который вывел следующие данные:
   -начальные затраты составили 0.00.., сообщаемая стоимость составила 5,04
   -общая стоимость выполнения запроса - 100
   -общее количество строк, которые доступны для возврата - 140
   -ожидаемое время выполнения запроса составило 0,337 мс, полученное время выполнения составило 0,363 мс
Благодаря индексу время выполнение запроса уменьшилось в 12 раз, а его общая стоимость уменьшилась в 333 раза!

### 3.1 Используйте команды EXPLAIN и ANALYZE, чтобы определить время и стоимость планирования, а также время и стоимость выполнения для выбора всех строк, где тема электронной почты — «Шокирующая экономия на праздничных покупках электрических самокатов» в первом запросе и «Черная пятница. Зеленые автомобили" во втором запросе.
```
EXPLAIN ANALYSE SELECT * FROM emails
WHERE email_subject LIKE 'Shocking Holiday Savings On Electric Scooters'
```
![image](https://github.com/user-attachments/assets/17450289-4405-4721-bebb-6a39db201067)

Запрос вывел следующие данные:
   -начальные затраты составили 0.00.., сообщаемая стоимость составила 10698,98
   -общая стоимость выполнения запроса - 20350
   -общее количество строк, которые доступны для возврата - 79
   -ожидаемое время выполнения запроса составило 0,256 мс, полученное время выполнения составило 158,608 мс

```
EXPLAIN ANALYSE SELECT * FROM emails
WHERE email_subject LIKE 'Black Friday. Green Cars.'
```
![image](https://github.com/user-attachments/assets/1e450a7a-8a85-45c9-896f-d6cab900ff64)

Запроскоторый вывел следующие данные:
   -начальные затраты составили 0.00.., сообщаемая стоимость составила 10698,98
   -общая стоимость выполнения запроса - 42150
   -общее количество строк, которые доступны для возврата - 79
   -ожидаемое время выполнения запроса составило 0,158 мс, полученное время выполнения составило 174,460 мс

### 3.2 Создайте хеш-сканирование в столбце email_subject
```
CREATE INDEX ix_email on emails using HASH(email_subject)
```

### 3.3 Повторите шаг 1. Сравните выходные данные планировщика запросов без хэш-индексас выходными данными с хэш-индексом. Как повлияло сканирование хэша на производительность двух запросов?
```
EXPLAIN ANALYSE SELECT * FROM emails
WHERE email_subject LIKE 'Shocking Holiday Savings On Electric Scooters'
```
![image](https://github.com/user-attachments/assets/90d5062d-d03d-4cdc-acc5-7bfa48e8b819)

Запрос вывел следующие данные:
   -начальные затраты составили 657,71.., сообщаемая стоимость составила 6384,09
   -общая стоимость выполнения запроса - 20350
   -общее количество строк, которые доступны для возврата - 79
   -ожидаемое время выполнения запроса составило 1,729 мс, полученное время выполнения составило 67,212 мс
Сканирование хэша помогло обеспечить более быстрое время выполнение запроса и низкую стоимость

```
EXPLAIN ANALYSE SELECT * FROM emails
WHERE email_subject LIKE 'Black Friday. Green Cars.'
```

Запрос вывел следующие данные:
   -начальные затраты составили 1358,66.., сообщаемая стоимость составила 7358,54
   -общая стоимость выполнения запроса - 42150
   -общее количество строк, которые доступны для возврата - 79
   -ожидаемое время выполнения запроса составило 0,260 мс, полученное время выполнения составило 39,030 мс
Сканирование хэша помогло обеспечить более быстрое время выполнение запроса и низкую стоимость

### 3.4 Создайте хэш-сканирование столбца customer_id.
```
CREATE INDEX ix_customer on emails using HASH(customer_id)
```

### 3.5 Используйте EXPLAIN и ANALYZE, чтобы оценить, сколько времени потребуется, чтобы выбрать все строки со значением customer_id больше 100. Какой тип сканирования использовался и почему?
```
EXPLAIN ANALYSE SELECT * FROM emails
WHERE customer_id > 100
```
![image](https://github.com/user-attachments/assets/1b03d6bc-3c6d-4def-b388-2c8c321e302e)

Запрос вывел следующие данные:
   -начальные затраты составили 0,00.., сообщаемая стоимость составила 10698,98
   -общая стоимость выполнения запроса - 417328
   -общее количество строк, которые доступны для возврата - 79
   -ожидаемое время выполнения запроса составило 0,180 мс, полученное время выполнения составило 145,312 мс
   -использовался Seq Scan так как HASH-индексы в PostgreSQL не поддерживают операции сравнения

## Вывод
Изучены методы оптимизации SQL-запросов для повышения производительности базы данных, включая анализ планов выполнения запросов с помощью EXPLAIN ANALYZE, применение различных типов индексов (B-tree, Hash, Bitmap), а также эффективное использование операций соединения (JOIN).

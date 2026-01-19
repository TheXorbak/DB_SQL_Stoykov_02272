# Лабораторная работа №4. Анализ производительности
### Цель:
Освоение методов анализа и оптимизации производительности БД.

### Результат:
Во время выполнения задания я глупо игнорировал пример написания генератора. Поэтому, выбрав путь немногим сложнее предложенного, проявил некоторую креативность в генерации записей.

### Генерация 20.000 записей в таблице guests
```sql
INSERT INTO stoykov02272.guests(id, name, birthday)
SELECT 
    id,
    (ARRAY['Ivanov', 'Smirnov', 'Kuznetsov', 'Popov', 'Vasilev', 
        'Petrov', 'Sokolov', 'Mikhaylov', 'Fedorov', 'Morozov',
        'Volkov', 'Alekseev', 'Lebedev', 'Semenov', 'Egorov', 
        'Pavlov', 'Kozlov', 'Stepanov', 'Nikolaev', 'Orlov'])[floor(random() * 20 + 1)] || ' ' ||
    (ARRAY['Aleksandr', 'Dmitriy', 'Maksim', 'Artem', 'Kirill', 
        'Ivan', 'Mikhail', 'Andrey', 'Aleksey', 'Ilya',
        'Sergey', 'Nikolay', 'Vladimir', 'Denis', 'Pavel', 
        'Anton', 'Igor', 'Oleg', 'Yuriy', 'Roman'])[floor(random() * 20 + 1)] AS name,
    CURRENT_DATE - 14*365 - (random() * 365 * 40)::int AS birthday
FROM generate_series(101, 20000) AS id
```

### Генерация 20.000 записей в таблице ordered_meal
```sql
INSERT INTO stoykov02272.ordered_meal(id, order_id, meal_id)
SELECT 
    i AS id,
  i - 6 AS order_id,
  CASE
    WHEN o.price = 500 THEN 1
    WHEN o.price = 749 THEN 2
    WHEN o.price = 579 THEN 3
    WHEN o.price = 899 THEN 4
    WHEN o.price = 159 THEN 5
    WHEN o.price = 499 THEN 6
  END AS meal_id
FROM generate_series(11, 20000) AS i
JOIN stoykov02272.orders o ON o.id = i - 6
```

### Генерация 20к записей в таблице orders.
В процессе выполнения задания была также расширена таблица tables (до 28 записей).
```sql
INSERT INTO stoykov02272.orders(id, price, date, table_id, guest_id)
SELECT
  id,
    (ARRAY[500, 749, 579, 899, 159, 499])[floor(random()*6+1)::int] AS price,
    CURRENT_DATE - (random() * 365 * 4)::int AS date,
  floor(random() * 28 + 1)::int AS table_id,
  id AS guest_id
FROM generate_series(5, 20000) AS id
```

### Анализ планов выполнения
Для анализа был взят запрос из 3 лабораторной работы:
```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT 
  o.id AS order_id,
  g.name AS guest_name,
  o.date AS order_date,
  o.price AS order_price,
  s.name AS waiter_name,
  m.name AS ordered_meals,
  m.price AS meal_price
  
FROM stoykov02272.orders AS o
JOIN stoykov02272.guests AS g ON guest_id = g.id
JOIN stoykov02272.tables AS t ON table_id = t.id
JOIN stoykov02272.staff AS s ON t.employee_id = s.id
JOIN stoykov02272.ordered_meal AS om ON o.id = om.order_id
JOIN stoykov02272.meals AS m ON om.meal_id = m.id
WHERE o.price BETWEEN 160 AND 898
ORDER BY order_date
```
### Анализ без индексов 
![image](/images/withoutindex01.jpg) ![image](/images/withoutindex02.jpg) 
### Анализ после создания индексов
Индексы ускорили выполнение запроса на 8мс. 
![image](/images/withindex01.jpg) ![image](/images/withindex02.jpg)
### Выводы
- Индексы ускоряют выполнение запросов с фильтрацией и JOIN
- Оптимизация особенно эффективна для аналитических запросов
- При больших объёмах данных индексация критически важна

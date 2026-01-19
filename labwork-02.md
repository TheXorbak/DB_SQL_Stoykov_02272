# Лабораторная работа №2. Инсталляция БД на сервере
### Цель:
Практическое развертывание базы данных и работа с SQL.
### Результат:
![image alt](https://github.com/TheXorbak/DB_SQL_Stoykov_02272/blob/main/myschema.png?raw=true)
К сожалению, CREATE запросов у меня не осталось, однако эта работа проверялась Кириллом Владимировичем

### Содержательный SELECT запрос:
```sql
SELECT 
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

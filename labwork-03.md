# Лабораторная работа №3. Представления и процедуры
### Цель:
Освоение механизмов абстракции данных и программных модулей.

### Результат:
Для выполнения задач и достижения цели был выбран 8 вариант: **Отдел кадров**.

Выбран и адаптирован под мою тему выходной документ: 
Выдать список сотрудников, имеющих более одного ребенка, сгруппированный по отделам с подсчетом среднего возраста сотрудников в каждом отделе.
### Создание функции
```sql
CREATE FUNCTION calc_average_age(post_id INT)
RETURNS INT AS $$
DECLARE res numeric;
BEGIN
  
  SELECT AVG(EXTRACT(YEAR FROM AGE(CURRENT_DATE, s.birthday))) INTO res
  FROM stoykov02272.staff s
  WHERE s.post_id = post_id;

  RETURN res;
END;
$$ LANGUAGE plpgsql;
```
**Назначение функции**

Созданная функция возвращает средний возраст переданного в параметр отдела. Использована в представлении.
### Создание представления
```sql
CREATE OR REPLACE VIEW stoykov02272.large_family_report AS
SELECT e.name, 
  p.name AS post,
  calc_average_age(p.id) AS average_post_age,
  e.sex, 
  e.birthday, 
  e.address, 
  CONCAT('******', SUBSTR(e.passport_details, 7, 10)) AS passport_details,
  e.children_number
FROM stoykov02272.staff e
JOIN stoykov02272.posts p ON e.post_id = p.id
WHERE e.children_number > 1
ORDER BY p.name;
```
**Назначение представления**

Представление выводит отчет о сотрудниках, имеющих более 1 ребёнка. Оно объединяет две таблицы и скрывает конфиденциальную информацию сотрудника.

**Использование представления**
```sql
SELECT * FROM stoykov02272.large_family_report
```

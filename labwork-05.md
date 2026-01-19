# Лабораторная работа №5. Триггеры и аудит
### Цель:
Реализация бизнес-логики на уровне БД и системы аудита.

### Создание журнала аудита
```sql
CREATE SEQUENCE stoykov02272.audit_log_audit_id_seq;

CREATE TABLE stoykov02272.audit_log (
      audit_id integer PRIMARY KEY NOT NULL DEFAULT nextval('stoykov02272.audit_log_audit_id_seq'::regclass),
      table_name VARCHAR(50) NOT NULL,
      operation_type VARCHAR(10) NOT NULL,
      operation_time TIMESTAMP
);
```

### Создание функции каскадного удаления таблицы
В данном случае мы рассматриваем удаление таблицы table, приводящее к удалению записей в связанной таблице orders (и записей связанной с таблицей orders таблицы ordered_meals)
```sql
CREATE OR REPLACE FUNCTION stoykov02272.cascade_delete_table()
RETURNS TRIGGER AS $$
BEGIN
      DELETE FROM stoykov02272.ordered_meal
      WHERE order_id IN (SELECT id FROM stoykov02272.orders WHERE table_id = OLD.id);

      DELETE FROM stoykov02272.orders
      WHERE table_id = OLD.id;

  INSERT INTO stoykov02272.audit_log (
          table_name,
          operation_type,
          operation_time
  )
      VALUES (
          'table',
          'DELETE',
          CURRENT_TIMESTAMP
  );

      RETURN OLD;
END; $$ LANGUAGE plpgsql;
```
**Триггер для этой функции**
```sql
CREATE TRIGGER trigger_log_table_deletions
BEFORE DELETE ON stoykov02272.tables
FOR EACH ROW
EXECUTE FUNCTION stoykov02272.cascade_delete_table();
```

### Создание функции аудита изменений
Используем для этой задачи таблицу staff
```sql
CREATE OR REPLACE FUNCTION stoykov02272.audit_staff_changes()
RETURNS TRIGGER AS $$
BEGIN
  IF (TG_OP = 'DELETE') THEN
          INSERT INTO stoykov02272.audit_log
          (table_name, operation_type, operation_type)
          VALUES
          ('staff', 'DELETE', CURRENT_TIMESTAMP);

      ELSIF (TG_OP = 'UPDATE') THEN
          INSERT INTO stoykov02272.audit_log
          (table_name, operation_type, operation_type)
          VALUES
          ('staff', 'UPDATE', CURRENT_TIMESTAMP);

      ELSIF (TG_OP = 'INSERT') THEN
          INSERT INTO stoykov02272.audit_log
          (table_name, operation_type, operation_type)
          VALUES
          ('staff', 'INSERT', CURRENT_TIMESTAMP);
      END IF;

      RETURN NULL; 
END; $$ LANGUAGE plpgsql
```
**Триггер для этой функции**
```sql
CREATE TRIGGER trigger_log_staff_changes
AFTER INSERT OR UPDATE OR DELETE
ON stoykov02272.staff
FOR EACH ROW
EXECUTE FUNCTION stoykov02272.audit_staff_changes();
```
### Проверка функций
**Удаление таблицы table:**
```sql
DELETE FROM stoykov02272.table
WHERE id = 10;
```
**Изменение таблицы staff:**
```sql
INSERT INTO stoykov02272.staff (id, name, post_id)
VALUES (13, 'Ivanov Petr Sergeevich', 3)
```
```sql
UPDATE stoykov02272.staff
SET post_id = 2
WHERE id = 13
```
```sql
DELETE FROM stoykov02272.staff
WHERE id = 13
```
**Результат запросов:**
```sql
SELECT * FROM stoykov02272.audit_log
```
![image](/images/mytestaudit.jpg)

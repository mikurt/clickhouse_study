# Репликация и другие фоновые процессы 

## 1. Возьмите любой демонстрационный DATASET

<img width="1554" height="763" alt="image" src="https://github.com/user-attachments/assets/6f3d5b5f-c1f9-4f43-b2d0-184b292e706b" />

## 2. Добавьте 2 реплики

## 3. Конвертируйте таблицу в реплицируемую, используя макрос replica.

## 4. Выполните запросы и отдайте результаты как 2 файла:
```
SELECT
getMacro(‘replica’),
*
FROM remote(’разделенный запятыми список реплик’,system.parts)
FORMAT JSONEachRow;
```

```
SELECT * FROM system.replicas FORMAT JSONEachRow;
```

## 5. Добавьте или выберите колонку с типом Date в таблице, добавьте TTL на таблицу «хранить последние 7 дней».
На проверку отправьте результат запроса «SHOW CREATE TABLE таблица»

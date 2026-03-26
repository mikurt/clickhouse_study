# Репликация и другие фоновые процессы 

## 1. Возьмите любой демонстрационный DATASET

<img width="1554" height="763" alt="image" src="https://github.com/user-attachments/assets/6f3d5b5f-c1f9-4f43-b2d0-184b292e706b" />

## 2. Добавьте 2 реплики

## 3. Конвертируйте таблицу в реплицируемую, используя макрос replica.

<img width="841" height="761" alt="image" src="https://github.com/user-attachments/assets/1779996c-24a6-4c73-ab46-5fd1d7ce7098" />

## 4. Выполните запросы и отдайте результаты как 2 файла:
```
SELECT
getMacro(‘replica’),
*
FROM remote(’разделенный запятыми список реплик’,system.parts)
FORMAT JSONEachRow;
```

<img width="1273" height="609" alt="image" src="https://github.com/user-attachments/assets/e09b390e-01fa-4f8e-a545-e3946dfa1cee" />

https://github.com/mikurt/clickhouse_study/blob/e6a091486513d4ce3cff8d5ec62fa7b8bfe63bb8/14/parts.result.ndjson

```
SELECT * FROM system.replicas FORMAT JSONEachRow;
```
<img width="1327" height="382" alt="image" src="https://github.com/user-attachments/assets/56360e90-aa63-4399-aba4-3fcbb5d35099" />

## 5. Добавьте или выберите колонку с типом Date в таблице, добавьте TTL на таблицу «хранить последние 7 дней».
На проверку отправьте результат запроса «SHOW CREATE TABLE таблица»

# Словари, оконные и табличные функции 
## 1. Создайте таблицу с полями:
```
user_id UInt64,
action String,
expense UInt64.
```

<img width="455" height="451" alt="image" src="https://github.com/user-attachments/assets/b01fdb58-13a8-42d7-9639-a22ebb74ec35" />

## 2. Создайте словарь, где ключ — user_id, атрибут — email (String), источник словаря выберите любой удобный, например, файл.

<img width="1252" height="614" alt="image" src="https://github.com/user-attachments/assets/b7539fff-a446-41ab-8679-a83551da7dad" />

## Наполните таблицу и источник данными с низкоардинальными значениями для поля action и несколькими повторяющимися строками для каждого user_id.

<img width="532" height="563" alt="image" src="https://github.com/user-attachments/assets/b2893f12-d226-46a4-b02d-192e1381b9a2" />

<img width="414" height="426" alt="image" src="https://github.com/user-attachments/assets/84fb20fc-31cb-4db9-90b2-7bbffbf0937c" />

## 3. Напишите SELECT, который возвращает:
- email с помощью dictGet,
- аккумулятивную сумму expense с окном по action,
- сортировку по email.

<img width="923" height="780" alt="image" src="https://github.com/user-attachments/assets/d688c310-c838-4358-b5b5-d2613a62495d" />

https://github.com/mikurt/clickhouse_study/blob/main/expenses_202603132322.csv

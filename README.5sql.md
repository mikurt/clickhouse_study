# Работа с SQL в ClickHouse
1. Создайте новую базу данных и перейдите в неё.
<img width="617" height="361" alt="image" src="https://github.com/user-attachments/assets/51d1ce1b-21ba-4ff0-8d4a-6e0d444e0504" />

2. Разработайте таблицу для бизнес-кейса "Меню ресторана" с минимум пятью полями. Наполните таблицу данными, используя модификаторы (например, Nullable, LowCardinality), где это необходимо. Не забудьте добавить комментарии к полям.
<img width="917" height="624" alt="image" src="https://github.com/user-attachments/assets/357b2b29-6fbb-492d-b479-1dfb119c065f" />
<img width="975" height="651" alt="image" src="https://github.com/user-attachments/assets/166dc9d4-bcbc-48e7-9541-d04c970d88f8" />
<img width="849" height="486" alt="image" src="https://github.com/user-attachments/assets/70260069-0090-460d-b12a-18e0c4ccffaa" />
<img width="1157" height="248" alt="image" src="https://github.com/user-attachments/assets/0ddb2270-bcb1-4be5-8f26-c25a51605b14" />

Заполнение таблицы более правдоподобными записями:

<img width="1112" height="699" alt="image" src="https://github.com/user-attachments/assets/ab7ffaf6-1a6c-4237-b11d-251211848d20" />

3. Протестируйте выполнение операций CRUD на созданной таблице.

5. Добавьте несколько новых полей в таблицу и удалите два-три существующих.
6. Выполните выборку данных (select) из любой таблицы из sample dataset
7. Материализуйте выбранную таблицу, создав её копию в виде отдельной таблицы.
8. Попрактикуйтесь с партициями: выполните операции ATTACH, DETACH и DROP. После этого добавьте новые данные в первоначально созданную таблицу.

# Мутации данных и манипуляции с партициями

## 1. Создание таблицы
Создайте таблицу user_activity с полями:

```
user_id (UInt32) — идентификатор пользователя
activity_type (String) — тип активности (например, 'login', 'logout', 'purchase')
activity_date (DateTime) — дата и время активности
```

Используйте MergeTree как движок таблицы и настройте партиционирование по дате активности (activity_date).

<img width="786" height="447" alt="image" src="https://github.com/user-attachments/assets/a6821cb5-0daf-418c-a05f-e55446ad7a45" />

## 2. Заполнение таблицы:
Вставьте несколько записей в таблицу user_activity. Используйте различные user_id, activity_type и activity_date.

<img width="1076" height="494" alt="image" src="https://github.com/user-attachments/assets/26333f7d-a7c3-42e5-a9a3-2f52c15d07c9" />

## 3. Выполнение мутаций:
Выполните мутацию для изменения типа активности у пользователя(-ей)

<img width="814" height="232" alt="image" src="https://github.com/user-attachments/assets/cb3e258e-9242-4430-a5a9-70d109999d24" />


## 4. Проверка результатов:
Напишите запрос для проверки изменений в таблице user_activity. Убедитесь, что тип активности у пользователей изменился. Приложите логи отслеживания мутаций в системной таблице.

<img width="835" height="263" alt="image" src="https://github.com/user-attachments/assets/6771d164-180e-452c-a331-9c2144f59097" />

<img width="931" height="502" alt="image" src="https://github.com/user-attachments/assets/5bf601dd-5779-4668-8cd7-905f8ce49357" />

## 5. Манипуляции с партициями:
Удалите партицию за определённый месяц.

<img width="553" height="206" alt="image" src="https://github.com/user-attachments/assets/2abb8c99-45b5-4480-a994-37de6aaeed6d" />

## 6. Проверка состояния таблицы:
Проверьте текущее состояние таблицы после удаления партиции. Убедитесь, что данные за указанный месяц были удалены.

<img width="734" height="268" alt="image" src="https://github.com/user-attachments/assets/7ca44a93-2f66-44ad-913e-dc97bb544063" />

## Дополнительные задания (по желанию):
Исследуйте, как работают другие типы мутаций.

<img width="780" height="224" alt="image" src="https://github.com/user-attachments/assets/80b26687-ad41-40be-937c-0a0cac54603a" />

<img width="713" height="247" alt="image" src="https://github.com/user-attachments/assets/01b6e884-06ca-4fcf-a5b6-a98958fa2687" />

<img width="781" height="502" alt="image" src="https://github.com/user-attachments/assets/f7b6744a-0c64-43c3-a95a-5f828a648acc" />

Изучите возможность использования TTL (Time to Live) для автоматического удаления старых партиций.

<img width="841" height="541" alt="image" src="https://github.com/user-attachments/assets/f9e23e80-d725-4def-97bd-36f1d8079164" />

<img width="718" height="233" alt="image" src="https://github.com/user-attachments/assets/04551a65-95e8-41d8-9166-9e1dbac46799" />


# UDF, Aggregate Functions and working with data types
## Вариант 1

Формируем данные

<img width="762" height="548" alt="image" src="https://github.com/user-attachments/assets/281d8b9a-8840-4f19-ad5e-c125708d1d52" />

<img width="971" height="443" alt="image" src="https://github.com/user-attachments/assets/6bc25472-4321-42da-947f-44f44756ff5b" />

- Агрегатные функции
  - Рассчитайте общий доход от всех операций.
 
<img width="606" height="189" alt="image" src="https://github.com/user-attachments/assets/3e920fa2-1547-468d-b4ec-32d171a50dc1" />

  - Найдите средний доход с одной сделки.

<img width="492" height="193" alt="image" src="https://github.com/user-attachments/assets/cfbda737-0ed3-4097-bb34-ec3e4fd7bab6" />

  - Определите общее количество проданной продукции.

<img width="518" height="181" alt="image" src="https://github.com/user-attachments/assets/0e608fd3-f807-4a0a-b10b-2fd61646bd7d" />

  - Подсчитайте количество уникальных пользователей, совершивших покупку.

<img width="653" height="200" alt="image" src="https://github.com/user-attachments/assets/f0707e1c-a627-494c-8991-5daa5538e520" />

- Функции для работы с типами данных
  - Преобразуйте `transaction_date` в строку формата `YYYY-MM-DD`.

<img width="1043" height="382" alt="image" src="https://github.com/user-attachments/assets/06080c30-9801-4d70-b1e3-2a6c46903bd7" />

  - Извлеките год и месяц из `transaction_date`.

<img width="830" height="345" alt="image" src="https://github.com/user-attachments/assets/b2b83d1b-1daf-4831-b701-e5dc11d4d84e" />

  - Округлите `price` до ближайшего целого числа.

<img width="720" height="397" alt="image" src="https://github.com/user-attachments/assets/4bbf3ae2-49da-49e1-be8a-93c02f94c3a0" />

  - Преобразуйте `transaction_id` в строку.

<img width="958" height="425" alt="image" src="https://github.com/user-attachments/assets/0f4b0ce5-b0c3-43d6-b78c-0a17cc256821" />

- User-Defined Functions (UDFs)
  - Создайте простую UDF для расчета общей стоимости транзакции.
  - Используйте созданную UDF для расчета общей цены для каждой транзакции.

<img width="833" height="437" alt="image" src="https://github.com/user-attachments/assets/4703c9a6-7826-42ea-840c-d6e158c28820" />

  - Создайте UDF для классификации транзакций на «высокоценные» и «малоценные» на основе порогового значения (например, 100).
  - Примените UDF для категоризации каждой транзакции.

<img width="865" height="428" alt="image" src="https://github.com/user-attachments/assets/15d4d961-59b2-4e1c-9f5d-6a76ba713352" />

# Движки MergeTree Family

## tbl1 - движок VersionedCollapsingMergeTree

<img width="1291" height="697" alt="image" src="https://github.com/user-attachments/assets/300fe733-ec86-4167-a894-cd5fd21dd113" />

<img width="673" height="546" alt="image" src="https://github.com/user-attachments/assets/fd157322-4f5a-4e56-9b91-273747ce6a21" />

Использует для удаления строки с противоположным знаком Sign и версию Version для определения актуальных записей движок VersionedCollapsingMergeTree

## tbl2 - движок SummingMergeTree

<img width="755" height="717" alt="image" src="https://github.com/user-attachments/assets/4ee36554-1b6a-406b-8c3a-014de2bb6bd6" />

Значения Value суммируются - движок SummingMergeTree

## tbl3 - движок ReplacingMergeTree

<img width="961" height="710" alt="image" src="https://github.com/user-attachments/assets/c950bd1c-c4b2-492c-9919-bf523f216511" />
<img width="803" height="562" alt="image" src="https://github.com/user-attachments/assets/17a839b3-1a65-4947-941d-1fe3a8c8cd4f" />

Служит для определения последней записи по первичному ключу - движок ReplacingMergeTree

## tbl4 - MergeTree

<img width="826" height="712" alt="image" src="https://github.com/user-attachments/assets/6b92cc20-d07e-4209-82b0-030ec4ba5e1d" />
<img width="561" height="270" alt="image" src="https://github.com/user-attachments/assets/00839944-efdb-4ba0-b461-ebf4e37f38e2" />

Условие не содержит значений таблицы после вставки. Исходя из определения таблицы, никаких особенностей не требуется, используем обычный MergeTree.

## tbl5 - AggregatingMergeTree

<img width="575" height="707" alt="image" src="https://github.com/user-attachments/assets/66924c08-a870-4157-9225-32c664f79431" />
<img width="868" height="791" alt="image" src="https://github.com/user-attachments/assets/b4536d1d-f59d-4061-8b02-b9406bed3736" />

Функция AggregateFunction используется в движке AggregatingMergeTree. Ошибка возникает из-за того, что во второй вставке для поля UserId надо использовать uniqState.

## tbl6 - CollapsingMergeTree

<img width="1011" height="756" alt="image" src="https://github.com/user-attachments/assets/02aee3ae-5778-4818-9ac2-40b53ba277f9" />
<img width="638" height="547" alt="image" src="https://github.com/user-attachments/assets/f0746e3b-264f-4be6-9757-056b90188ea3" />

Использование поля sign для удаления предыдущих записей с тем же ключом - движок CollapsingMergeTree

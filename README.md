# clickhouse_study

## Примеры использования ClickHouse в компаниях

- Яндекс использует для анализа кликов на сайтах
- Наша компания использует в качестве быстрых аналитических витрин данных для анализа, а также для накопления и анализа тегов производственного оборудования

## Специфика проекта

Крупная золотодобывающая компания. Clickhouse используем в качестве быстрых аналитических витрин данных для анализа. Витрина предварительно подготавливается (слои STG, ODS, DDS, DM) в Greenplum. Готовык витрины и спарвоники переносятся в Clickhouse. Для интеграции используем движок Postgresql. Оркестрация выполняется Airflow. 

На текущий момент Clickhouse используется однонодовый, вторая нода для реплики.

## Установка
ВМ в яндекс облаке:

<img width="616" height="838" alt="image" src="https://github.com/user-attachments/assets/62cc20db-9368-4ada-9b54-828c18a19a83" />

Запущенный Clickhouse

<img width="1510" height="364" alt="image" src="https://github.com/user-attachments/assets/fb4fe30d-39cd-4bc3-a21a-5e8029d78f23" />

Загружен датасет:

<img width="1341" height="843" alt="image" src="https://github.com/user-attachments/assets/82876e5a-ed55-4585-9311-2c29bbab36e9" />

Кол-во записей:

<img width="1051" height="320" alt="image" src="https://github.com/user-attachments/assets/7b4f0349-d447-463a-a51d-77615afc3430" />

Подключение в DBeaver:

<img width="497" height="335" alt="image" src="https://github.com/user-attachments/assets/252cfa65-3386-4717-a328-0b84f578fdc6" />


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

## Тестирование производительности

Тестирую запросом *clickhouse-benchmark --concurrency 10 --timelimit 60 <<< "SELECT pickup_ntaname, count(distinct dropoff_ntaname), sum(passenger_count),     avg(trip_distance),     avg(fare_amount),    avg(tip_amount),    avg(tolls_amount),    avg(total_amount),    max(payment_type) from trips group by pickup_ntaname"*

Результат при начальной настройке:

<img width="1091" height="401" alt="image" src="https://github.com/user-attachments/assets/40349ea9-18b5-4343-ac5c-9ba974c3f564" />

Результат после донастройки:

<img width="1094" height="406" alt="image" src="https://github.com/user-attachments/assets/20295b3b-2133-4d89-bba1-222f087b2f22" />

Получен прирост производительности на 3%.

## Примененные настройки

### config.d/x_config.xml

- общую память ограничил 80% от 16 Гб:
```
    <max_server_memory_usage>13000000000</max_server_memory_usage>
```
- лимит на всю память процесса ClickHouse:
```
    <max_server_memory_usage_fraction>0.9</max_server_memory_usage_fraction>
```
- Буфер меток хранит индексы в памяти. Для 16 ГБ RAM 5 ГБ:
```
    <mark_cache_size>5368709120</mark_cache_size>
```

- Буфер несжатых данных ускоряет повторные запросы к одним и тем же колонкам:
```
    <uncompressed_cache_size>2147483648</uncompressed_cache_size>
```
- Ограничение размера слияний и ускорение записи на диск:
```
  <merge_tree>
        <max_bytes_to_merge_at_min_space_in_pool>1073741824</max_bytes_to_merge_at_min_space_in_pool> <!-- 1 ГБ -->
        <max_bytes_to_merge_at_max_space_in_pool>16106127360</max_bytes_to_merge_at_max_space_in_pool> <!-- 15 ГБ -->
        <min_rows_for_wide_part>1048576</min_rows_for_wide_part>
        <min_bytes_for_wide_part>268435456</min_bytes_for_wide_part>
    </merge_tree>
```
- Приоритет выполнения. Если CPU перегружен, снижаем приоритет фоновых задач относительно запросов SELECT:
```
    <background_merges_mutations_concurrency_ratio>2</background_merges_mutations_concurrency_ratio>
```
- Сжатие по умолчанию:
```
    <compression>
        <case>
            <method>zstd</method>
            <level>3</level> <!-- Уровни выше 3 сильно грузят CPU при записи -->
        </case>
    </compression>
```
### users.d/x_users.xml

- Максимум потоков по числу ядер:
```
            <max_threads>4</max_threads>
```
- Максимум потоков для вставки
```
            <max_insert_threads>2</max_insert_threads>
```
- Сбрасывать данные на диск, если не хватает RAM для GROUP BY:
```
            <max_bytes_before_external_group_by>13000000000</max_bytes_before_external_group_by>
```
- Сбрасывать на диск для SORT:
```
            <max_bytes_before_external_sort>13000000000</max_bytes_before_external_sort>
```
- Не давать одному потоку резервировать слишком много памяти:
```
            <memory_usage_overcommit_ratio_denominator>10</memory_usage_overcommit_ratio_denominator>
```

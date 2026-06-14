# Интеграция с Apache Kafka
## 1. Установите Apache Kafka удобным способом

<img width="923" height="814" alt="image" src="https://github.com/user-attachments/assets/f148a187-e543-4755-97d3-7e44e90537c3" />

Тест:
<img width="941" height="618" alt="image" src="https://github.com/user-attachments/assets/ffaad265-f64a-4901-8ff0-7a3566d117ea" />

## 2. Установите ClickHouse и выполните необходимые настройки для работы с Kafka
nano /etc/clickhouse-server/config.d/kafka.xml
```xml
    <clickhouse>
        <kafka>
            <sasl_username>tmyu</sasl_username>
            <sasl_password>kafkatmyu</sasl_password>
            <security_protocol>sasl_ssl</security_protocol>
            <sasl_mechanism>SCRAM-SHA-512</sasl_mechanism>
            <enable_ssl_certificate_verification>false</enable_ssl_certificate_verification>
        </kafka>
    </clickhouse>
```

nano /etc/clickhouse-server/config.xml
<img width="1044" height="743" alt="image" src="https://github.com/user-attachments/assets/f1c1bfaf-10d6-42f2-8678-6f635058253f" />


## 3. Запишите данные в Kafka и настройте пайплайн чтения через Kafka Engine, затем переместите их в таблицу MergeTree с помощью Materialized View

Создание таблицы Kafka, обычной таблицы и материализованного вью для автоматического сохранения из Kafka в таблицу.
<img width="921" height="749" alt="image" src="https://github.com/user-attachments/assets/b8ffa812-ad9f-4e33-bf8a-81de93283672" />

## 4. Проверьте корректность чтения данных из Kafka в ClickHouse

Записываю данные в Kafka:

<img width="985" height="524" alt="image" src="https://github.com/user-attachments/assets/a7e28a16-f724-470c-8f1b-91a4a80b5ea2" />

Данные пришли в Clickhouse:
<img width="1413" height="412" alt="image" src="https://github.com/user-attachments/assets/c9bc7461-902f-4c6d-a8bf-eba0201d94ed" />

Еще 1 сообщение:
<img width="846" height="204" alt="image" src="https://github.com/user-attachments/assets/10ba3b94-9af6-40b1-842d-e38325332cbb" />

Обновленная таблица:

<img width="705" height="323" alt="image" src="https://github.com/user-attachments/assets/300d0f42-0e59-4c5d-a0c5-b64da28882ad" />





# 15 Шардирование и распределенные запросы
## Настройка кластера и топологий

- Запустите N экземпляров clickhouse-server.
- Опишите две или более топологий объединения экземпляров в шарды, указав фактор репликации и количество шардов.
- Предоставьте xml-секцию в текстовом файле для проверки.

Есть три сервера. Создал 2 топологии:
- 3 шарда cl_3sh
- двухуровневая: 1-й уровень cl_lev1, 2-й уровень - cl_lev2

См. 

## Создать DISTRIBUTED-таблицу на каждую из топологий. 

Можно использовать системную таблицу system.one, содержащую одну колонку dummy типа UInt8, в качестве локальной таблицы.

или 5) предоставить вывод запроса SELECT *,hostName(),_shard_num from distributed-table для каждой distributed-таблицы, можно добавить group by и limit по вкусу 
если тестовых данных много.

### Топология "3 шарда"

<img width="843" height="607" alt="image" src="https://github.com/user-attachments/assets/09df62eb-2805-4171-9f73-4a5b2dc6a779" />

### Топология "2 уровня"

<img width="618" height="424" alt="image" src="https://github.com/user-attachments/assets/943789da-202e-48d9-a162-24c5057e638d" />

<img width="838" height="479" alt="image" src="https://github.com/user-attachments/assets/e7cc27ec-0aef-403d-88b9-456c0a4766c3" />

### предоставить SELECT * FROM system.clusters

<img width="1280" height="450" alt="image" src="https://github.com/user-attachments/assets/ec57b22c-2b8d-43b4-aa61-050863ff910e" />


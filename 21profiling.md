# Профилирование запросов

## 1. Возьмите любой демонстрационный DATASET

```sql
CREATE TABLE youtube
(
`id` String,
`fetch_date` DateTime,
`upload_date_str` String,
`upload_date` Date,
`title` String,
`uploader_id` String,
`uploader` String,
`uploader_sub_count` Int64,
`is_age_limit` Bool,
`view_count` Int64,
`like_count` Int64,
`dislike_count` Int64,
`is_crawlable` Bool,
`has_subtitles` Bool,
`is_ads_enabled` Bool,
`is_comments_enabled` Bool,
`description` String,
`rich_metadata` Array(Tuple(call String, content String, subtitle String, title String, url String)),
`super_titles` Array(Tuple(text String, url String)),
`uploader_badges` String,
`video_badges` String
)
ENGINE = MergeTree
PARTITION BY toStartOfMonth(upload_date)
ORDER BY (uploader_id, upload_date)

INSERT INTO youtube
SETTINGS input_format_null_as_default = 1
SELECT
id,
parseDateTimeBestEffortUSOrZero(toString(fetch_date)) AS fetch_date,
upload_date AS upload_date_str,
toDate(parseDateTimeBestEffortUSOrZero(upload_date::String)) AS upload_date,
ifNull(title, '') AS title,
uploader_id,
ifNull(uploader, '') AS uploader,
uploader_sub_count,
is_age_limit,
view_count,
like_count,
dislike_count,
is_crawlable,
has_subtitles,
is_ads_enabled,
is_comments_enabled,
ifNull(description, '') AS description,
rich_metadata,
super_titles,
ifNull(uploader_badges, '') AS uploader_badges,
ifNull(video_badges, '') AS video_badges
FROM s3(
'https://clickhouse-public-datasets.s3.amazonaws.com/youtube/original/files/*.zst',
'JSONLines'
)
WHERE year(upload_date) IN (2020, 2021)
LIMIT 100000000

```

<img width="443" height="192" alt="image" src="https://github.com/user-attachments/assets/2dcb705b-d194-4b67-b073-8748e253ffb3" />

Примечание: датасет недогрузился, буду тестировать на загруженных 61 млн.

## 2. Выполните два запроса
- Один с условием WHERE, не использующим первичный ключ (например, с фильтрацией по другому столбцу).

```sql
SELECT countIf(uploader_id, like_count>1000) as liked_uploaders FROM default.youtube
where toDate(fetch_date) = '2021-11-28' SETTINGS send_logs_level = 'trace', use_query_cache = 0;
```
- Другой с условием WHERE, использующим первичный ключ (например, с фильтрацией по столбцу, являющемуся PK).

```sql
SELECT countIf(like_count>40) as liked_videos FROM default.youtube
where uploader_id in ('SC-rCcF_9OFfx7D-GFZSBqPg', 'SC4xC2bjhAFuHs551N0fXaHA', 'SC7bBukApl2XfRfesr33DNHg')  SETTINGS send_logs_level = 'trace', use_query_cache = 0;
```

## 3. Сравните текстовые логи запросов. Выделите строки, относящиеся к пробегу основного индекса в логах запросов.

Запрос 1:
```sql
SELECT countIf(uploader_id, like_count > 1000) AS liked_uploaders
FROM default.youtube
WHERE toDate(fetch_date) = '2021-11-28'
SETTINGS send_logs_level = 'trace', use_query_cache = 0
```
> Query id: 03feb850-f073-4d3f-a5b7-98284118cf32

> [clickhouse-node] 2026.04.26 14:23:24.438276 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> executeQuery: (from 127.0.0.1:51212) (query 1, line 1) SELECT countIf(uploader_id, like_count>1000) as liked_uploaders FROM default.youtube where toDate(fetch_date) = '2021-11-28' SETTINGS send_logs_level = 'trace', use_query_cache = 0; (stage: Complete)
> 
> [clickhouse-node] 2026.04.26 14:23:24.439385 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Planner: Query to stage Complete
> 
> [clickhouse-node] 2026.04.26 14:23:24.439550 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Planner: Query from stage FetchColumns to stage Complete
> 
> [clickhouse-node] 2026.04.26 14:23:24.439737 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Aggregator: Adjusting memory limit before external aggregation with 5.83 GiB (ratio: 0.5, available system memory: 11.66 GiB)
> 
> [clickhouse-node] 2026.04.26 14:23:24.440038 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b): Loading statistics
> 
> [clickhouse-node] 2026.04.26 14:23:24.443898 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> QueryPlanOptimizePrewhere: The min valid primary key position for moving to the tail of PREWHERE is -1
> 
> [clickhouse-node] 2026.04.26 14:23:24.443945 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> QueryPlanOptimizePrewhere: Moved 1 conditions to PREWHERE
> 
> [clickhouse-node] 2026.04.26 14:23:24.444196 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> IInterpreterUnionOrSelectQuery: The new analyzer is enabled, but the old interpreter is used. It can be a bug, please report it. Will disable 'allow_experimental_analyzer' setting (for query: SELECT min(upload_date), max(upload_date), min(uploader_id), max(uploader_id), count() GROUP BY toStartOfMonth(upload_date) SETTINGS aggregate_functions_null_for_empty = false, transform_null_in = false, legacy_column_name_of_tuple_literal = false)
> 
> **[clickhouse-node] 2026.04.26 14:23:24.444731 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Key condition: unknown**
> 
> [clickhouse-node] 2026.04.26 14:23:24.445133 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): MinMax index condition: unknown
> 
> [clickhouse-node] 2026.04.26 14:23:24.445581 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Query condition cache has dropped 2303/7579 granules for PREWHERE condition equals(toDate(__table1.fetch_date), '2021-11-28'_String).
> 
> [clickhouse-node] 2026.04.26 14:23:24.445620 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Query condition cache has dropped 0/5276 granules for WHERE condition equals(toDate(fetch_date), '2021-11-28'_String).
> 
> [clickhouse-node] 2026.04.26 14:23:24.445652 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Filtering marks by primary and secondary keys
> 
> **[clickhouse-node] 2026.04.26 14:23:24.446733 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): PK index has dropped 0/5276 granules, it took 0ms across 4 threads.**
> 
> [clickhouse-node] 2026.04.26 14:23:24.446821 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Selected 183/183 parts by partition key, 39 parts by primary key, **5276/7579 marks by primary key**, 5276 marks to read from 39 ranges
> 
> [clickhouse-node] 2026.04.26 14:23:24.446880 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Spreading mark ranges among streams (default reading)
> 
> [clickhouse-node] 2026.04.26 14:23:24.447815 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Reading approx. 42735436 rows with 4 streams
> 
> [clickhouse-node] 2026.04.26 14:23:24.478791 [ 17626 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregating
> 
> [clickhouse-node] 2026.04.26 14:23:24.478868 [ 17626 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> HashTablesStatistics: An entry for key=16556096684112032227 found in cache: sum_of_sizes=4, median_size=1
> 
> [clickhouse-node] 2026.04.26 14:23:24.478892 [ 17626 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Aggregator: Aggregation method: without_key
> 
> [clickhouse-node] 2026.04.26 14:23:24.489424 [ 17682 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregating
> 
> [clickhouse-node] 2026.04.26 14:23:24.489499 [ 17682 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> HashTablesStatistics: An entry for key=16556096684112032227 found in cache: sum_of_sizes=4, median_size=1
> 
> [clickhouse-node] 2026.04.26 14:23:24.489530 [ 17682 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Aggregator: Aggregation method: without_key
> 
> [clickhouse-node] 2026.04.26 14:23:24.494603 [ 17666 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregating
> 
> [clickhouse-node] 2026.04.26 14:23:24.494696 [ 17666 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> HashTablesStatistics: An entry for key=16556096684112032227 found in cache: sum_of_sizes=4, median_size=1
> 
> [clickhouse-node] 2026.04.26 14:23:24.494754 [ 17666 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Aggregator: Aggregation method: without_key
> 
> [clickhouse-node] 2026.04.26 14:23:24.555287 [ 17604 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregating
> 
> [clickhouse-node] 2026.04.26 14:23:24.555358 [ 17604 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> HashTablesStatistics: An entry for key=16556096684112032227 found in cache: sum_of_sizes=4, median_size=1
> 
> [clickhouse-node] 2026.04.26 14:23:24.555384 [ 17604 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Aggregator: Aggregation method: without_key
> 
> [clickhouse-node] 2026.04.26 14:23:28.108283 [ 17682 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregated. 2182981 to 1 rows (from 68.70 MiB) in 3.660137598 sec. (596420.474 rows/sec., 18.77 MiB/sec.)
> 
> [clickhouse-node] 2026.04.26 14:23:28.115765 [ 17604 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregated. 2130867 to 1 rows (from 67.06 MiB) in 3.667619054 sec. (580994.637 rows/sec., 18.28 MiB/sec.)
> 
> [clickhouse-node] 2026.04.26 14:23:28.137923 [ 17666 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregated. 628533 to 1 rows (from 19.78 MiB) in 3.689767392 sec. (170344.884 rows/sec., 5.36 MiB/sec.)
> 
> [clickhouse-node] 2026.04.26 14:23:28.141048 [ 17626 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> AggregatingTransform: Aggregated. 2514322 to 1 rows (from 79.13 MiB) in 3.692897429 sec. (680853.462 rows/sec., 21.43 MiB/sec.)
> 
> [clickhouse-node] 2026.04.26 14:23:28.141110 [ 17626 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Trace> Aggregator: Merging aggregated data
> 
> [clickhouse-node] 2026.04.26 14:23:28.141739 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> MemoryTracker: Query peak memory usage: 18.01 MiB.
> 
> [clickhouse-node] 2026.04.26 14:23:28.143280 [ 16951 ] {03feb850-f073-4d3f-a5b7-98284118cf32} <Debug> executeQuery: Read 42735436 rows, 1.55 GiB in 3.705061 sec., 11534340.73015262 rows/sec., 429.00 MiB/sec.

```
   ┌─liked_uploaders─┐
1. │          236147 │
   └─────────────────┘

1 row in set. Elapsed: 3.704 sec. Processed 42.74 million rows, 1.67 GB (11.54 million rows/s., 449.93 MB/s.)
Peak memory usage: 18.01 MiB.
```

Запрос2:
```sql
SELECT countIf(like_count > 40) AS liked_videos
FROM default.youtube
WHERE uploader_id IN ('SC-rCcF_9OFfx7D-GFZSBqPg', 'SC4xC2bjhAFuHs551N0fXaHA', 'SC7bBukApl2XfRfesr33DNHg')
SETTINGS send_logs_level = 'trace', use_query_cache = 0
```

> Query id: 02ef2b2e-0213-47cc-b3c8-df87b6e51f4f

> [clickhouse-node] 2026.04.26 14:40:16.421866 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> executeQuery: (from 127.0.0.1:51212) (query 1, line 1) SELECT countIf(like_count>40) as liked_videos FROM default.youtube where uploader_id in ('SC-rCcF_9OFfx7D-GFZSBqPg', 'SC4xC2bjhAFuHs551N0fXaHA', 'SC7bBukApl2XfRfesr33DNHg') SETTINGS send_logs_level = 'trace', use_query_cache = 0; (stage: Complete)
>
> [clickhouse-node] 2026.04.26 14:40:16.423151 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> Planner: Query to stage Complete
>
> [clickhouse-node] 2026.04.26 14:40:16.423350 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> Planner: Query from stage FetchColumns to stage Complete
>
> [clickhouse-node] 2026.04.26 14:40:16.423538 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> Aggregator: Adjusting memory limit before external aggregation with 5.83 GiB (ratio: 0.5, available system memory: 11.66 GiB)
>
> [clickhouse-node] 2026.04.26 14:40:16.423878 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b): Loading statistics
>
> [clickhouse-node] 2026.04.26 14:40:16.427610 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> QueryPlanOptimizePrewhere: The min valid primary key position for moving to the tail of PREWHERE is 0
>
> [clickhouse-node] 2026.04.26 14:40:16.427648 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> QueryPlanOptimizePrewhere: Moved 1 conditions to PREWHERE
>
> [clickhouse-node] 2026.04.26 14:40:16.427910 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> IInterpreterUnionOrSelectQuery: The new analyzer is enabled, but the old interpreter is used. It can be a bug, please report it. Will disable 'allow_experimental_analyzer' setting (for query: SELECT min(upload_date), max(upload_date), min(uploader_id), max(uploader_id), count() GROUP BY toStartOfMonth(upload_date) SETTINGS aggregate_functions_null_for_empty = false, transform_null_in = false, legacy_column_name_of_tuple_literal = false)
>
> **[clickhouse-node] 2026.04.26 14:40:16.428469 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Key condition: (column 0 in 3-element set)**
>
> [clickhouse-node] 2026.04.26 14:40:16.428796 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): MinMax index condition: unknown
>
> [clickhouse-node] 2026.04.26 14:40:16.429027 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Query condition cache has dropped 29/7579 granules for PREWHERE condition in(__table1.uploader_id, __set_String_4062025620592713759_13964588722198947060).
>
> [clickhouse-node] 2026.04.26 14:40:16.429415 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Query condition cache has dropped 7547/7550 granules for WHERE condition in(uploader_id, __set_String_4062025620592713759_13964588722198947060).
>
> [clickhouse-node] 2026.04.26 14:40:16.429442 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Filtering marks by primary and secondary keys
>
> [clickhouse-node] 2026.04.26 14:40:16.429654 [ 17628 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Used generic exclusion search over index for part 20200101_1270_4104_3 with 1 steps
>
> [clickhouse-node] 2026.04.26 14:40:16.429694 [ 17607 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Used generic exclusion search over index for part 20200101_5456_6069_2 with 1 steps
>
> [clickhouse-node] 2026.04.26 14:40:16.429698 [ 17629 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Used generic exclusion search over index for part 20200101_6219_6341_1 with 1 steps
>
> **[clickhouse-node] 2026.04.26 14:40:16.429890 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): PK index has dropped 137/140 granules, it took 0ms across 3 threads.**
>
> [clickhouse-node] 2026.04.26 14:40:16.429945 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Selected 183/183 parts by partition key, 3 parts by primary key, **3/7579 marks by primary key**, 3 marks to read from 3 ranges
>
> [clickhouse-node] 2026.04.26 14:40:16.429979 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Spreading mark ranges among streams (default reading)
>
> [clickhouse-node] 2026.04.26 14:40:16.430147 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> default.youtube (cd5b5184-0c50-4c86-bc3b-5c291047ea5b) (SelectExecutor): Reading approx. 24576 rows with 3 streams
>
> [clickhouse-node] 2026.04.26 14:40:16.432868 [ 17611 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> AggregatingTransform: Aggregating
>
> [clickhouse-node] 2026.04.26 14:40:16.432918 [ 17611 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> HashTablesStatistics: An entry for key=1226326657430025890 found in cache: sum_of_sizes=4, median_size=1
>
> [clickhouse-node] 2026.04.26 14:40:16.432946 [ 17611 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> Aggregator: Aggregation method: without_key
>
> [clickhouse-node] 2026.04.26 14:40:16.433067 [ 17611 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> AggregatingTransform: Aggregated. 2 to 1 rows (from 2.00 B) in 0.002673567 sec. (748.064 rows/sec., 748.06 B/sec.)
>
> [clickhouse-node] 2026.04.26 14:40:16.433531 [ 17679 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> AggregatingTransform: Aggregating
>
> [clickhouse-node] 2026.04.26 14:40:16.433577 [ 17679 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> HashTablesStatistics: An entry for key=1226326657430025890 found in cache: sum_of_sizes=4, median_size=1
>
> [clickhouse-node] 2026.04.26 14:40:16.433612 [ 17679 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> Aggregator: Aggregation method: without_key
>
> [clickhouse-node] 2026.04.26 14:40:16.433720 [ 17624 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> AggregatingTransform: Aggregating
>
> [clickhouse-node] 2026.04.26 14:40:16.433729 [ 17679 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> AggregatingTransform: Aggregated. 8 to 1 rows (from 8.00 B) in 0.003334247 sec. (2399.342 rows/sec., 2.34 KiB/sec.)
>
> [clickhouse-node] 2026.04.26 14:40:16.433756 [ 17624 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> HashTablesStatistics: An entry for key=1226326657430025890 found in cache: sum_of_sizes=4, median_size=1
>
> [clickhouse-node] 2026.04.26 14:40:16.433806 [ 17624 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> Aggregator: Aggregation method: without_key
>
> [clickhouse-node] 2026.04.26 14:40:16.433895 [ 17624 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> AggregatingTransform: Aggregated. 1 to 1 rows (from 1.00 B) in 0.003514368 sec. (284.546 rows/sec., 284.55 B/sec.)
>
> [clickhouse-node] 2026.04.26 14:40:16.433913 [ 17624 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Trace> Aggregator: Merging aggregated data
>
> [clickhouse-node] 2026.04.26 14:40:16.434536 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> MemoryTracker: Query peak memory usage: 1.76 MiB.
>
> [clickhouse-node] 2026.04.26 14:40:16.435234 [ 16951 ] {02ef2b2e-0213-47cc-b3c8-df87b6e51f4f} <Debug> executeQuery: Read 24576 rows, 840.00 KiB in 0.013469 sec., 1824634.3455341896 rows/sec., 60.90 MiB/sec.

```
   ┌─liked_videos─┐
1. │            2 │
   └──────────────┘

1 row in set. Elapsed: 0.013 sec. Processed 24.58 thousand rows, 860.16 KB (1.82 million rows/s., 63.82 MB/s.)
Peak memory usage: 1.76 MiB.
```

Вывод: в первом запросе первичный ключ почти не помог, читалось 43 млн строк. Во вотором запросе первичный ключ эффективно отсек лишее, читалось всего 25 тыс строк.

## 4. Используйте команду EXPLAIN для анализа выполнения запросов, покажите использование индекса в выводе.

Запрос 1:
```
EXPLAIN indexes = 1
SELECT countIf(uploader_id, like_count > 1000) AS liked_uploaders
FROM default.youtube
WHERE toDate(fetch_date) = '2021-11-28'
SETTINGS use_query_cache = 0

Query id: faf1a682-2696-4d6d-9fe1-407f62174f0d

    ┌─explain────────────────────────────────────────────────────────────────┐
 1. │ Expression ((Project names + Projection))                              │
 2. │   Aggregating                                                          │
 3. │     Expression (Before GROUP BY)                                       │
 4. │       Expression ((WHERE + Change column names to column identifiers)) │
 5. │         ReadFromMergeTree (default.youtube)                            │
 6. │         Indexes:                                                       │
 7. │           MinMax                                                       │
 8. │             Condition: true                                            │
 9. │             Parts: 175/175                                             │
10. │             Granules: 7582/7582                                        │
11. │           Partition                                                    │
12. │             Condition: true                                            │
13. │             Parts: 175/175                                             │
14. │             Granules: 7582/7582                                        │
15. │           PrimaryKey                                                   │
16. │             Condition: true                                            │
17. │             Parts: 175/175                                             │
18. │             Granules: 7582/7582                                        │
19. │           Ranges: 175                                                  │
    └────────────────────────────────────────────────────────────────────────┘

19 rows in set. Elapsed: 0.012 sec.
```

Запрос 2:
```
EXPLAIN indexes = 1
SELECT countIf(like_count > 1000) AS liked_videos
FROM default.youtube
WHERE uploader_id IN ('SC-rCcF_9OFfx7D-GFZSBqPg', 'SC4xC2bjhAFuHs551N0fXaHA', 'SC7bBukApl2XfRfesr33DNHg')
SETTINGS use_query_cache = 0

Query id: 616837b2-676b-4df6-9d99-4870ee19bb17

    ┌─explain────────────────────────────────────────────────────────────────┐
 1. │ Expression ((Project names + Projection))                              │
 2. │   Aggregating                                                          │
 3. │     Expression (Before GROUP BY)                                       │
 4. │       Expression ((WHERE + Change column names to column identifiers)) │
 5. │         ReadFromMergeTree (default.youtube)                            │
 6. │         Indexes:                                                       │
 7. │           MinMax                                                       │
 8. │             Condition: true                                            │
 9. │             Parts: 175/175                                             │
10. │             Granules: 7582/7582                                        │
11. │           Partition                                                    │
12. │             Condition: true                                            │
13. │             Parts: 175/175                                             │
14. │             Granules: 7582/7582                                        │
15. │           PrimaryKey                                                   │
16. │             Keys:                                                      │
17. │               uploader_id                                              │
18. │             Condition: (uploader_id in 3-element set)                  │
19. │             Parts: 32/175                                              │
20. │             Granules: 32/7582                                          │
21. │             Search Algorithm: generic exclusion search                 │
22. │           Ranges: 32                                                   │
    └────────────────────────────────────────────────────────────────────────┘

22 rows in set. Elapsed: 0.017 sec.
```



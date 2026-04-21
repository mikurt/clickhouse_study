# Мониторинг

- Придумайте несколько запросов для персонализированного мониторинга.
- Создайте таблицу с этими запросами в нужном формате
- На проверку отправьте скриншот встроенного дашборда, показывающего эти таблицы

Таблица метрик и ее заполнение:

```sql
CREATE or replace TABLE user_stats
(
    user String,
    event_ts DateTime,
    exec_queries AggregateFunction(count, UInt64),
    avg_duration AggregateFunction(avg, UInt64),
    max_memory AggregateFunction(max, UInt64),
    rows_read AggregateFunction(sum, UInt64),
    exec_errors AggregateFunction(countIf, UInt8)
)
ENGINE = AggregatingMergeTree()
ORDER BY (event_ts, user);

CREATE MATERIALIZED VIEW user_stats_mv
TO user_stats
AS SELECT
    user,
    toStartOfMinute(event_time) AS event_ts,
    countState(*) AS exec_queries,
    avgState(query_duration_ms) AS avg_duration,
    maxState(memory_usage) AS max_memory,
    sumState(read_rows) AS rows_read,
    countIfState(type = 'ExceptionWhileProcessing') AS exec_errors
FROM system.query_log
WHERE type IN ('QueryFinish', 'ExceptionWhileProcessing')
GROUP BY user, event_ts;
```

Запросы для монгиторинга во встроенном дашборде:

```sql
WITH toDateTimeOrDefault({from:String}, '', now() - {seconds:UInt32}) AS from,
    toDateTimeOrDefault({to:String}, '', now()) AS to,
    intDiv({rounding:UInt32}+59, 60)*60 as adj_rounding,
    (SELECT groupArray(user) FROM (
        SELECT user FROM default.user_stats 
        WHERE event_ts BETWEEN from AND to
        GROUP BY user ORDER BY countMerge(exec_queries) DESC LIMIT 4
    )) AS top_users
SELECT toStartOfInterval(event_ts, INTERVAL adj_rounding SECOND)::INT AS t,
    if(has(top_users, user), user, 'Others') AS user,
    countMerge(exec_queries)
  FROM default.user_stats
  WHERE event_ts BETWEEN from AND to
group by all
ORDER by t WITH FILL STEP adj_rounding;

WITH toDateTimeOrDefault({from:String}, '', now() - {seconds:UInt32}) AS from,
    toDateTimeOrDefault({to:String}, '', now()) AS to,
    intDiv({rounding:UInt32}+59, 60)*60 as adj_rounding,
    (SELECT groupArray(user) FROM (
        SELECT user FROM default.user_stats 
        WHERE event_ts BETWEEN from AND to
        GROUP BY user ORDER BY avgMerge(avg_duration) DESC LIMIT 4
    )) AS top_users
SELECT toStartOfInterval(event_ts, INTERVAL adj_rounding SECOND)::INT AS t,
    if(has(top_users, user), user, 'Others') AS user,
    avgMerge(avg_duration)
  FROM default.user_stats
  WHERE event_ts BETWEEN from AND to
group by all
ORDER by t WITH FILL STEP adj_rounding;

WITH toDateTimeOrDefault({from:String}, '', now() - {seconds:UInt32}) AS from,
    toDateTimeOrDefault({to:String}, '', now()) AS to,
    intDiv({rounding:UInt32}+59, 60)*60 as adj_rounding,
    (SELECT groupArray(user) FROM (
        SELECT user FROM default.user_stats 
        WHERE event_ts BETWEEN from AND to
        GROUP BY user ORDER BY maxMerge(max_memory) DESC LIMIT 4
    )) AS top_users
SELECT toStartOfInterval(event_ts, INTERVAL adj_rounding SECOND)::INT AS t,
    if(has(top_users, user), user, 'Others') AS user,
    maxMerge(max_memory)
  FROM default.user_stats
  WHERE event_ts BETWEEN from AND to
group by all
ORDER by t WITH FILL STEP adj_rounding;

WITH toDateTimeOrDefault({from:String}, '', now() - {seconds:UInt32}) AS from,
    toDateTimeOrDefault({to:String}, '', now()) AS to,
    intDiv({rounding:UInt32}+59, 60)*60 as adj_rounding,
    (SELECT groupArray(user) FROM (
        SELECT user FROM default.user_stats 
        WHERE event_ts BETWEEN from AND to
        GROUP BY user ORDER BY sumMerge(rows_read) DESC LIMIT 4
    )) AS top_users
SELECT toStartOfInterval(event_ts, INTERVAL adj_rounding SECOND)::INT AS t,
    if(has(top_users, user), user, 'Others') AS user,
    sumMerge(rows_read)
  FROM default.user_stats
  WHERE event_ts BETWEEN from AND to
group by all
ORDER by t WITH FILL STEP adj_rounding;

WITH toDateTimeOrDefault({from:String}, '', now() - {seconds:UInt32}) AS from,
    toDateTimeOrDefault({to:String}, '', now()) AS to,
    intDiv({rounding:UInt32}+59, 60)*60 as adj_rounding,
    (SELECT groupArray(user) FROM (
        SELECT user FROM default.user_stats 
        WHERE event_ts BETWEEN from AND to
        GROUP BY user ORDER BY countIfMerge(exec_errors) DESC LIMIT 4
    )) AS top_users
SELECT toStartOfInterval(event_ts, INTERVAL adj_rounding SECOND)::INT AS t,
    if(has(top_users, user), user, 'Others') AS user,
    countIfMerge(exec_errors)
  FROM default.user_stats
  WHERE event_ts BETWEEN from AND to
group by all
ORDER by t WITH FILL STEP adj_rounding;
```

Дашборд:
<img width="1597" height="733" alt="image" src="https://github.com/user-attachments/assets/b9aeb0a6-565b-44ff-bb79-03209dfbba69" />

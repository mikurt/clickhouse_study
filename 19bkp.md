# Storage Policy и резервное копирование
- Разверните S3 с использованием MinIO, Ceph или Object Storage от Yandex Cloud.

<img width="1368" height="443" alt="image" src="https://github.com/user-attachments/assets/99b9cd13-c0f8-4512-8571-666035e35a09" />

- Установите clickhouse-backup и настройте политику хранения (storage policy) в конфигурации ClickHouse.

<img width="1596" height="673" alt="image" src="https://github.com/user-attachments/assets/70e6dfd8-778f-439b-a875-e90b3c3c1d16" />

Config.xml для clickhouse-backup:

```yml
general:
    remote_storage: s3
    max_file_size: 0
    backups_to_keep_local: 0
    backups_to_keep_remote: 0
    log_level: info
    allow_empty_backups: false
    download_concurrency: 4
    upload_concurrency: 4
    upload_max_bytes_per_second: 0
    download_max_bytes_per_second: 0
    object_disk_server_side_copy_concurrency: 32
    allow_object_disk_streaming: false
    use_resumable_state: true
    restore_schema_on_cluster: ""
    upload_by_part: true
    download_by_part: true
    restore_database_mapping: {}
    restore_table_mapping: {}
    retries_on_failure: 3
    retries_pause: 5s
    retries_jitter: 0
    watch_interval: 1h
    full_interval: 24h
    watch_backup_name_template: shard{shard}-{type}-{time:20060102150405}
    sharded_operation_mode: ""
    cpu_nice_priority: 15
    io_nice_priority: idle
    rbac_backup_always: true
    rbac_conflict_resolution: recreate
    config_backup_always: false
    named_collections_backup_always: false
    delete_batch_size: 1000
    retriesduration: 5s
    watchduration: 1h0m0s
    fullduration: 24h0m0s
clickhouse:
    username: default
    password: "chtmyu"
    host: localhost
    port: 9000
    disk_mapping: {}
    skip_tables:
        - system.*
        - INFORMATION_SCHEMA.*
        - information_schema.*
        - _temporary_and_external_tables.*
    skip_table_engines: []
    skip_disks: []
    skip_disk_types: []
    timeout: 30m
    freeze_by_part: false
    freeze_by_part_where: ""
    use_embedded_backup_restore: false
    embedded_backup_disk: ""
    backup_mutations: true
    restore_as_attach: false
    restore_distributed_cluster: ""
    check_parts_columns: true
    secure: false
    skip_verify: false
    sync_replicated_tables: false
    log_sql_queries: true
    config_dir: /etc/clickhouse-server/
    restart_command: exec:systemctl restart clickhouse-server
    ignore_not_exists_error_during_freeze: true
    check_replicas_before_attach: true
    default_replica_path: /clickhouse/tables/{cluster}/{shard}/{database}/{table}
    default_replica_name: '{replica}'
    tls_key: ""
    tls_cert: ""
    tls_ca: ""
    max_connections: 2
    debug: false
s3:
    access_key: "YCAJE0esJXz8JpQbC59ey-BsD"
    secret_key: "YCOPvRXea47VZgy_p4BwP2UnwIQHyNGh7DCGKyNP"
    bucket: "s3-ch-mikurt1"
    endpoint: "http://storage.yandexcloud.net"
    acl: private
    assume_role_arn: ""
    force_path_style: false
    path: "backup/{cluster}/{shard}"
    object_disk_path: "object_disk/{cluster}/{shard}"
    disable_ssl: false
    compression_level: 1
    compression_format: tar
    sse: ""
    sse_kms_key_id: ""
    sse_customer_algorithm: ""
    sse_customer_key: ""
    sse_customer_key_md5: ""
    sse_kms_encryption_context: ""
    disable_cert_verification: false
    use_custom_storage_class: false
    storage_class: STANDARD
    custom_storage_class_map: {}
    concurrency: 3
    max_parts_count: 4000
    allow_multipart_download: false
    object_labels: {}
    request_payer: ""
    check_sum_algorithm: ""
    request_content_md5: false
    retry_mode: standard
    chunk_size: 5242880
    delete_concurrency: 10
    debug: false
```

Политика хранения (storage policy) в конфигурации ClickHouse:

/etc/clickhouse-server/config.d/storage.xml

```xml
<clickhouse>
        <storage_configuration>
            <disks>
                <default>
                </default>
                <s3_cold>
                    <type>s3</type>
                    <endpoint>http://storage.yandexcloud.net/s3-ch-mikurt1/cold_data/</endpoint>
                    <access_key_id>YCAJE0esJXz8JpQbC59ey-BsD</access_key_id>
                    <secret_access_key>YCOPvRXea47VZgy_p4BwP2UnwIQHyNGh7DCGKyNP</secret_access_key>
                </s3_cold>
            </disks>
            <policies>
                <hot_cold>
                    <volumes>
                        <hot>
                            <disk>default</disk>
                        </hot>
                        <cold>
                            <disk>s3_cold</disk>
                        </cold>
                    </volumes>
                    <move_factor>0.1</move_factor>
                </hot_cold>
            </policies>
        </storage_configuration>
</clickhouse>
```

<img width="622" height="453" alt="image" src="https://github.com/user-attachments/assets/76d23e85-fe73-4005-82b2-e1d0ff46baac" />

<img width="545" height="363" alt="image" src="https://github.com/user-attachments/assets/f045bab8-57ab-41d0-ad3a-02f9113c8643" />

- Создайте тестовую базу данных с несколькими таблицами и заполните их данными.

<img width="393" height="531" alt="image" src="https://github.com/user-attachments/assets/34a0cbb4-d90d-41bf-a297-b3068195ed28" />

- Выполните резервное копирование на удалённый ресурс (S3).

<img width="1589" height="817" alt="image" src="https://github.com/user-attachments/assets/b0f428c2-ee9e-4b22-90d9-337da29faeb1" />

<img width="1594" height="453" alt="image" src="https://github.com/user-attachments/assets/a83d167d-5e1d-411a-b83b-8381fb272768" />

<img width="1198" height="424" alt="image" src="https://github.com/user-attachments/assets/f226aa98-7c78-4ab5-a173-522ca61423a5" />

- Повредите данные (удалите таблицу, измените строки и т.д.)

<img width="747" height="242" alt="image" src="https://github.com/user-attachments/assets/760b798c-2204-4475-a102-0e74fef128e4" />

- Восстановите данные из резервной копии.

<img width="1589" height="431" alt="image" src="https://github.com/user-attachments/assets/a8110722-1349-4d4a-a34e-8a15ab0d7ad9" />

```
yc-user@clickhouse-node:/etc/clickhouse-backup$ sudo -u clickhouse clickhouse-backup restore_remote backup_default_s3_1 -t default.tbl1 --rm
2026-04-05 21:18:41.135 INF pkg/clickhouse/clickhouse.go:120 > clickhouse connection success: tcp://localhost:9000
2026-04-05 21:18:41.135 INF pkg/clickhouse/clickhouse.go:1185 > SELECT value FROM `system`.`build_options` where name='VERSION_INTEGER'
2026-04-05 21:18:41.138 INF pkg/clickhouse/clickhouse.go:1185 > SELECT countIf(name='type') AS is_disk_type_present, countIf(name='object_storage_type') AS is_object_storage_type_present, countIf(name='free_space') AS is_free_space_present, countIf(name='disks') AS is_storage_policy_present FROM system.columns WHERE database='system' AND table IN ('disks','storage_policies')
2026-04-05 21:18:41.144 INF pkg/clickhouse/clickhouse.go:1185 > SELECT d.path AS path, any(d.name) AS name, any(lower(if(d.type='ObjectStorage',d.object_storage_type,d.type))) AS type, min(d.free_space) AS free_space, groupUniqArray(s.policy_name) AS storage_policies FROM system.disks AS d  LEFT JOIN (SELECT policy_name, arrayJoin(disks) AS disk FROM system.storage_policies) AS s ON s.disk = d.name GROUP BY d.path
2026-04-05 21:18:41.151 INF pkg/clickhouse/clickhouse.go:322 > clickhouse connection closed
2026-04-05 21:18:41.152 INF pkg/clickhouse/clickhouse.go:120 > clickhouse connection success: tcp://localhost:9000
2026-04-05 21:18:41.152 INF pkg/clickhouse/clickhouse.go:1185 > SELECT countIf(name='type') AS is_disk_type_present, countIf(name='object_storage_type') AS is_object_storage_type_present, countIf(name='free_space') AS is_free_space_present, countIf(name='disks') AS is_storage_policy_present FROM system.columns WHERE database='system' AND table IN ('disks','storage_policies')
2026-04-05 21:18:41.159 INF pkg/clickhouse/clickhouse.go:1185 > SELECT d.path AS path, any(d.name) AS name, any(lower(if(d.type='ObjectStorage',d.object_storage_type,d.type))) AS type, min(d.free_space) AS free_space, groupUniqArray(s.policy_name) AS storage_policies FROM system.disks AS d  LEFT JOIN (SELECT policy_name, arrayJoin(disks) AS disk FROM system.storage_policies) AS s ON s.disk = d.name GROUP BY d.path
2026-04-05 21:18:41.166 INF pkg/clickhouse/clickhouse.go:1185 > SELECT count() AS is_macros_exists FROM system.tables WHERE database='system' AND name='macros'  SETTINGS empty_result_for_aggregation_by_empty_set=0
2026-04-05 21:18:41.170 INF pkg/clickhouse/clickhouse.go:1185 > SELECT macro, substitution FROM system.macros
2026-04-05 21:18:41.172 INF pkg/clickhouse/clickhouse.go:1185 > SELECT count() AS is_macros_exists FROM system.tables WHERE database='system' AND name='macros'  SETTINGS empty_result_for_aggregation_by_empty_set=0
2026-04-05 21:18:41.176 INF pkg/clickhouse/clickhouse.go:1185 > SELECT macro, substitution FROM system.macros
2026-04-05 21:18:41.189 INF pkg/resumable/state.go:102 > parameters changed old=map[string]interface {}{"dataOnly":false, "dropExists":false, "needClean":"true.17468657431691270162", "partitions":[]interface {}{}, "schemaOnly":false, "tablePattern":"default.tbl1"} new=map[string]interface {}{"dataOnly":false, "dropExists":true, "needClean":"true.15730474648572895389", "partitions":[]string{}, "schemaOnly":false, "tablePattern":"default.tbl1"}, /var/lib/clickhouse/backup/backup_default_s3_1/restore.state2 cleanup begin
2026-04-05 21:18:41.214 INF pkg/clickhouse/clickhouse.go:1183 > SELECT name, count(*) as is_present FROM system.settings WHERE name IN (?, ?) GROUP BY name with args []interface {}{"display_secrets_in_show_and_select", "show_table_uuid_in_table_create_query_if_not_nil"}
2026-04-05 21:18:41.221 INF pkg/clickhouse/clickhouse.go:1185 > SELECT name FROM system.databases WHERE engine IN ('MySQL','PostgreSQL','MaterializedPostgreSQL')
2026-04-05 21:18:41.223 INF pkg/clickhouse/clickhouse.go:1185 >    SELECT     countIf(name='data_path') is_data_path_present,     countIf(name='data_paths') is_data_paths_present,     countIf(name='uuid') is_uuid_present,     countIf(name='create_table_query') is_create_table_query_present,     countIf(name='total_bytes') is_total_bytes_present    FROM system.columns WHERE database='system' AND table='tables'
2026-04-05 21:18:41.249 INF pkg/clickhouse/clickhouse.go:1185 > SELECT database, name, engine , data_paths , uuid , create_table_query , coalesce(total_bytes, 0) AS total_bytes   FROM system.tables WHERE is_temporary = 0 AND match(concat(database,'.',name),'^default\.tbl1$')  ORDER BY total_bytes DESC SETTINGS show_table_uuid_in_table_create_query_if_not_nil=1
2026-04-05 21:18:41.303 INF pkg/clickhouse/clickhouse.go:1185 > SELECT data_path AS metadata_path FROM system.databases WHERE name = 'system' LIMIT 1
2026-04-05 21:18:41.306 INF pkg/clickhouse/clickhouse.go:1185 > DROP TABLE IF EXISTS `default`.`tbl1` NO DELAY
2026-04-05 21:18:41.307 INF pkg/clickhouse/clickhouse.go:1185 > CREATE DATABASE IF NOT EXISTS `default` ENGINE=Atomic
2026-04-05 21:18:41.308 INF pkg/clickhouse/clickhouse.go:1185 > CREATE TABLE default.tbl1 UUID '719ca171-3399-49b5-8c01-998a3e9bd8d4' (`UserID` UInt64, `PageViews` UInt8, `Duration` UInt8, `Sign` Int8, `Version` UInt8) ENGINE = VersionedCollapsingMergeTree(Sign, Version) ORDER BY UserID SETTINGS index_granularity = 8192
2026-04-05 21:18:41.322 INF pkg/backup/restore.go:1481 > done, backup=backup_default_s3_1, duration=16ms, operation=restore_schema
2026-04-05 21:18:41.323 INF pkg/clickhouse/clickhouse.go:1183 > SELECT name, count(*) as is_present FROM system.settings WHERE name IN (?, ?) GROUP BY name with args []interface {}{"show_table_uuid_in_table_create_query_if_not_nil", "display_secrets_in_show_and_select"}
2026-04-05 21:18:41.330 INF pkg/clickhouse/clickhouse.go:1185 > SELECT name FROM system.databases WHERE engine IN ('MySQL','PostgreSQL','MaterializedPostgreSQL')
2026-04-05 21:18:41.332 INF pkg/clickhouse/clickhouse.go:1185 >    SELECT     countIf(name='data_path') is_data_path_present,     countIf(name='data_paths') is_data_paths_present,     countIf(name='uuid') is_uuid_present,     countIf(name='create_table_query') is_create_table_query_present,     countIf(name='total_bytes') is_total_bytes_present    FROM system.columns WHERE database='system' AND table='tables'
2026-04-05 21:18:41.338 INF pkg/clickhouse/clickhouse.go:1185 > SELECT database, name, engine , data_paths , uuid , create_table_query , coalesce(total_bytes, 0) AS total_bytes   FROM system.tables WHERE is_temporary = 0 AND match(concat(database,'.',name),'^default\.tbl1$')  ORDER BY total_bytes DESC SETTINGS show_table_uuid_in_table_create_query_if_not_nil=1
2026-04-05 21:18:41.389 INF pkg/clickhouse/clickhouse.go:1185 > SELECT data_path AS metadata_path FROM system.databases WHERE name = 'system' LIMIT 1
2026-04-05 21:18:41.392 INF pkg/clickhouse/clickhouse.go:1185 > SELECT sum(bytes_on_disk) as size FROM system.parts WHERE active AND database='default' AND table='tbl1' GROUP BY database, table
2026-04-05 21:18:41.398 INF pkg/backup/restore.go:2143 > download object_disks start, table=default.tbl1
2026-04-05 21:18:41.398 INF pkg/backup/restore.go:2150 > download object_disks finish, duration=0s, size=0B, database=default, table=tbl1
2026-04-05 21:18:41.398 INF pkg/clickhouse/clickhouse.go:1185 > ALTER TABLE `default`.`tbl1` ATTACH PART 'all_1_1_1'
2026-04-05 21:18:41.400 INF pkg/clickhouse/clickhouse.go:1185 > ALTER TABLE `default`.`tbl1` ATTACH PART 'all_2_2_1'
2026-04-05 21:18:41.402 INF pkg/backup/restore.go:2076 > done, database=default, duration=5ms, operation=restoreDataRegular, progress=1/1, table=tbl1
2026-04-05 21:18:41.402 INF pkg/backup/restore.go:1943 > done, backup=backup_default_s3_1, operation=restore_data, duration=80ms
2026-04-05 21:18:41.402 INF pkg/clickhouse/clickhouse.go:1185 > DROP FUNCTION IF EXISTS `total_value`
2026-04-05 21:18:41.403 INF pkg/clickhouse/clickhouse.go:1185 > CREATE OR REPLACE FUNCTION total_value AS (qty, price) -> (qty * price)
2026-04-05 21:18:41.408 INF pkg/clickhouse/clickhouse.go:1185 > DROP FUNCTION IF EXISTS `h_fields`
2026-04-05 21:18:41.409 INF pkg/clickhouse/clickhouse.go:1185 > CREATE OR REPLACE FUNCTION h_fields AS h -> CAST((groupConcatArray(';')(arrayReverse(h)), h[-1], h[-2], h[-3]), 'Tuple(path String, l1 String, l2 String, l3 String)')
2026-04-05 21:18:41.414 INF pkg/clickhouse/clickhouse.go:1185 > DROP FUNCTION IF EXISTS `hier`
2026-04-05 21:18:41.415 INF pkg/clickhouse/clickhouse.go:1185 > CREATE OR REPLACE FUNCTION hier AS (dict, k) -> h_fields([dictGet(dict, 'email', k), dictGet(dict, 'email', k - 1), dictGet(dict, 'email', k - 2)])
2026-04-05 21:18:41.419 INF pkg/clickhouse/clickhouse.go:1185 > DROP FUNCTION IF EXISTS `value_level`
2026-04-05 21:18:41.420 INF pkg/clickhouse/clickhouse.go:1185 > CREATE OR REPLACE FUNCTION value_level AS (q, p) -> if(total_value(q, p) >= 100, 'высокоценные', 'малоценные')
2026-04-05 21:18:41.433 INF pkg/backup/restore.go:313 > done, duration=281ms, operation=restore, version=2.6.43
2026-04-05 21:18:41.433 INF pkg/clickhouse/clickhouse.go:322 > clickhouse connection closed
```

- Убедитесь, что повреждённые данные успешно восстановлены.

<img width="778" height="259" alt="image" src="https://github.com/user-attachments/assets/960e25c0-49b6-4a43-9763-13660d3ccb5c" />


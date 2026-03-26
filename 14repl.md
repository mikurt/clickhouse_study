# Репликация и другие фоновые процессы 

## 1. Возьмите любой демонстрационный DATASET

<img width="1554" height="763" alt="image" src="https://github.com/user-attachments/assets/6f3d5b5f-c1f9-4f43-b2d0-184b292e706b" />

## 2. Добавьте 2 реплики

/etc/clickhouse-server/config.d/remote.xml

```
<clickhouse>
    <remote_servers>
        <!-- Test only shard config for testing distributed storage -->
        <ch_replica_rs>
            <!-- Inter-server per-cluster secret for Distributed queries
                 default: no secret (no authentication will be performed)

                 If set, then Distributed queries will be validated on shards, so at least:
                 - such cluster should exist on the shard,
                 - such cluster should have the same secret.

                 And also (and which is more important), the initial_user will
                 be used as current user for the query.

                 Right now the protocol is pretty simple, and it only takes into account:
                 - cluster name
                 - query

                 Also, it will be nice if the following will be implemented:
                 - source hostname (see interserver_http_host), but then it will depend on DNS,
                   it can use IP address instead, but then you need to get correct on the initiator node.
                 - target hostname / ip address (same notes as for source hostname)
                 - time-based security tokens
            -->
            <!-- <secret></secret> -->

            <shard>
                <!-- Optional. Whether to write data to just one of the replicas. Default: false (write data to all replicas). -->
                <!-- <internal_replication>false</internal_replication> -->
                <!-- Optional. Shard weight when writing data. Default: 1. -->
                <!-- <weight>1</weight> -->
                <replica>
                    <host>10.130.0.18</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>chtmyu</password>
                    <!-- Optional. Priority of the replica for load_balancing. Default: 1 (less value has more priority). -->
                    <!-- <priority>1</priority> -->
                    <!-- Use SSL? Default: no -->
                    <!-- <secure>0</secure> -->
                    <!-- Optional. Bind to specific host before connecting to use a specific network. -->
                    <!-- <bind_host>10.0.0.1</bind_host> -->
                </replica>
                <replica>
                    <host>10.130.0.29</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>chtmyu</password>
                    <!-- Optional. Priority of the replica for load_balancing. Default: 1 (less value has more priority). -->
                    <!-- <priority>1</priority> -->
                    <!-- Use SSL? Default: no -->
                    <!-- <secure>0</secure> -->
                    <!-- Optional. Bind to specific host before connecting to use a specific network. -->
                    <!-- <bind_host>10.0.0.1</bind_host> -->
                </replica>

            </shard>
        </ch_replica_rs>
    </remote_servers>
    <!-- Substitutions for parameters of replicated tables.
          Optional. If you don't use replicated tables, you could omit that.

         See https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/replication/#creating-replicated-tables
      -->
    <macros>
        <shard>01</shard>
        <replica>main</replica>
    </macros>

</clickhouse>
```

/etc/clickhouse-server/config.d/keeper.xml

```
<clickhouse>
    <!-- ZooKeeper is used to store metadata about replicas, when using Replicated tables.
         Optional. If you don't use replicated tables, you could omit that.

         See https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/replication/

         Note: ClickHouse Cloud https://clickhouse.com/cloud has ClickHouse Keeper automatically configured for every service.
      -->


    <zookeeper>
        <node>
            <host>10.130.0.18</host>
            <port>9181</port>
        </node>
<!--
        <node>
            <host>example2</host>
            <port>2181</port>
        </node>
        <node>
            <host>example3</host>
            <port>2181</port>
        </node>
-->
    </zookeeper>


</clickhouse>
```

/etc/clickhouse-keeper/keeper_config.xml

```
<clickhouse>
    <listen_host>0.0.0.0</listen_host>
    <logger>
        <!-- Possible levels [1]:

          - none (turns off logging)
          - fatal
          - critical
          - error
          - warning
          - notice
          - information
          - debug
          - trace

            [1]: https://github.com/pocoproject/poco/blob/poco-1.9.4-release/Foundation/include/Poco/Logger.h#L105-L114
        -->
        <level>trace</level>
        <log>/var/log/clickhouse-keeper/clickhouse-keeper.log</log>
        <errorlog>/var/log/clickhouse-keeper/clickhouse-keeper.err.log</errorlog>
        <!-- Rotation policy
             See https://github.com/pocoproject/poco/blob/poco-1.9.4-release/Foundation/include/Poco/FileChannel.h#L54-L85
          -->
        <size>1000M</size>
        <count>10</count>
        <!-- <console>1</console> --> <!-- Default behavior is autodetection (log to console if not daemon mode and is tty) -->
    </logger>

    <max_connections>4096</max_connections>

    <keeper_server>
            <tcp_port>9181</tcp_port>

            <!-- Must be unique among all keeper serves -->
            <server_id>1</server_id>

            <log_storage_path>/var/lib/clickhouse/coordination/logs</log_storage_path>
            <snapshot_storage_path>/var/lib/clickhouse/coordination/snapshots</snapshot_storage_path>

            <coordination_settings>
                <operation_timeout_ms>10000</operation_timeout_ms>
                <min_session_timeout_ms>10000</min_session_timeout_ms>
                <session_timeout_ms>100000</session_timeout_ms>
                <raft_logs_level>information</raft_logs_level>
                <compress_logs>false</compress_logs>
                <!-- All settings listed in https://github.com/ClickHouse/ClickHouse/blob/master/src/Coordination/CoordinationSettings.h -->
            </coordination_settings>

            <!-- enable sanity hostname checks for cluster configuration (e.g. if localhost is used with remote endpoints) -->
            <hostname_checks_enabled>true</hostname_checks_enabled>
            <raft_configuration>
                <server>
                    <id>1</id>

                    <!-- Internal port and hostname -->
                    <hostname>10.130.0.18</hostname>
                    <port>9234</port>
                </server>

                <!-- Add more servers here -->

            </raft_configuration>
    </keeper_server>

    <openSSL>
      <server>            <!-- Used for secure tcp port -->
            <!-- openssl req -subj "/CN=localhost" -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout /etc/clickhouse-server/server.key -out /etc/clickhouse-server/server.>
            <!-- <certificateFile>/etc/clickhouse-keeper/server.crt</certificateFile> -->
            <!-- <privateKeyFile>/etc/clickhouse-keeper/server.key</privateKeyFile> -->
            <!-- dhparams are optional. You can delete the <dhParamsFile> element.
                 To generate dhparams, use the following command:
                  openssl dhparam -out /etc/clickhouse-keeper/dhparam.pem 4096
                 Only file format with BEGIN DH PARAMETERS is supported.
              -->
            <!-- <dhParamsFile>/etc/clickhouse-keeper/dhparam.pem</dhParamsFile> -->
            <verificationMode>none</verificationMode>
            <loadDefaultCAFile>true</loadDefaultCAFile>
            <cacheSessions>true</cacheSessions>
            <disableProtocols>sslv2,sslv3</disableProtocols>
            <preferServerCiphers>true</preferServerCiphers>
        </server>
    </openSSL>

</clickhouse>

```


## 3. Конвертируйте таблицу в реплицируемую, используя макрос replica.

<img width="841" height="761" alt="image" src="https://github.com/user-attachments/assets/1779996c-24a6-4c73-ab46-5fd1d7ce7098" />

## 4. Выполните запросы и отдайте результаты как 2 файла:
```
SELECT
getMacro(‘replica’),
*
FROM remote(’разделенный запятыми список реплик’,system.parts)
FORMAT JSONEachRow;
```

<img width="1273" height="609" alt="image" src="https://github.com/user-attachments/assets/e09b390e-01fa-4f8e-a545-e3946dfa1cee" />

https://github.com/mikurt/clickhouse_study/blob/e6a091486513d4ce3cff8d5ec62fa7b8bfe63bb8/14/parts.result.ndjson

```
SELECT * FROM system.replicas FORMAT JSONEachRow;
```

<img width="1327" height="382" alt="image" src="https://github.com/user-attachments/assets/56360e90-aa63-4399-aba4-3fcbb5d35099" />

https://github.com/mikurt/clickhouse_study/blob/ac9ea4672463459270fed3ecd5e3973a97a2af9f/14/replicas.result.ndjson

## 5. Добавьте или выберите колонку с типом Date в таблице, добавьте TTL на таблицу «хранить последние 7 дней».
На проверку отправьте результат запроса «SHOW CREATE TABLE таблица»

<img width="1178" height="691" alt="image" src="https://github.com/user-attachments/assets/2f55b8b8-4c23-4a08-8550-b5066bbaa90e" />

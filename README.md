# Роль ansible для разворачивания Percona XtraDB Cluster

За основу взята роль [https://github.com/cdelgehier/ansible-role-XtraDB-Cluster](https://github.com/cdelgehier/ansible-role-XtraDB-Cluster ) и немного актуализирована.

В каталоге [roles](roles/) (на тестовом хосте путь /home/otus/ansible/xtradb) находятся роли:
-  [01_tuning_OS](roles/01_tuning_OS) - Предварительная настройка ОС с последующей перезагрузкой.
- [02_ansible-role-XtraDB-Cluster](roles/02_ansible-role-XtraDB-Cluster) - Установка и настройка percona xtradb cluster.
- [create-otus-xtradb-cluster](roles/create-otus-xtradb-cluster.yml) - Роль объединяющая в себе 01_tuning_OS и 02_ansible-role-XtraDB-Cluster, но почему-то падает в середине выполнения второй роли. Не стал разбираться с причинами.
- [delete_percona](roles/delete_percona) - Удаление со всеми хвостами percona xtradb cluster, может пригодится для последующей повторной установки.


Переменные, необходимые(желаемые) для установки кластера percona можно определить в файле [02_ansible-role-XtraDB-Cluster.yml](roles/02_ansible-role-XtraDB-Cluster.yml). При этом, описание переменных дано в файле [02_ansible-role-XtraDB-Cluster/README.md](roles/02_ansible-role-XtraDB-Cluster/README.md)

Собственно, установка кластера запускается от имени пользователя otus таким образом:
```bash
ansible-playbook 02_ansible-role-XtraDB-Cluster.yml -i 02_ansible-role-XtraDB-Cluster/hosts
```

В результате установки кластера создаётся тестовая БД otusdb.

Так же создаются пользователи с паролями (можно переопределить в через переменные):
```mysql
'root'@'localhost' - Root!19Pass 
'otus'@'%' - Otus!19Pass (привилегии: 'otusdb.*:ALL')
```
#### Проверяем статусы на разных нодах:

```mysql
mysql> SHOW STATUS LIKE 'wsrep_local_state_comment';
+---------------------------+--------+
| Variable_name             | Value  |
+---------------------------+--------+
| wsrep_local_state_comment | Synced |
+---------------------------+--------+
1 row in set (0.01 sec)
```
```mysql
mysql> show global status like 'wsrep_cluster_size';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+
1 row in set (0.01 sec)
```
```mysql
mysql> show global status like 'wsrep%';
+----------------------------------+----------------------------------------------------+
| Variable_name                    | Value                                              |
+----------------------------------+----------------------------------------------------+
| wsrep_local_state_uuid           | c09d5a38-d233-11e9-9d3a-2ae4d93e72bc               |
| wsrep_protocol_version           | 9                                                  |
| wsrep_last_applied               | 14                                                 |
| wsrep_last_committed             | 14                                                 |
| wsrep_replicated                 | 0                                                  |
| wsrep_replicated_bytes           | 0                                                  |
| wsrep_repl_keys                  | 0                                                  |
| wsrep_repl_keys_bytes            | 0                                                  |
| wsrep_repl_data_bytes            | 0                                                  |
| wsrep_repl_other_bytes           | 0                                                  |
| wsrep_received                   | 11                                                 |
| wsrep_received_bytes             | 2157                                               |
| wsrep_local_commits              | 0                                                  |
| wsrep_local_cert_failures        | 0                                                  |
| wsrep_local_replays              | 0                                                  |
| wsrep_local_send_queue           | 0                                                  |
| wsrep_local_send_queue_max       | 1                                                  |
| wsrep_local_send_queue_min       | 0                                                  |
| wsrep_local_send_queue_avg       | 0.000000                                           |
| wsrep_local_recv_queue           | 0                                                  |
| wsrep_local_recv_queue_max       | 1                                                  |
| wsrep_local_recv_queue_min       | 0                                                  |
| wsrep_local_recv_queue_avg       | 0.000000                                           |
| wsrep_local_cached_downto        | 7                                                  |
| wsrep_flow_control_paused_ns     | 0                                                  |
| wsrep_flow_control_paused        | 0.000000                                           |
| wsrep_flow_control_sent          | 0                                                  |
| wsrep_flow_control_recv          | 0                                                  |
| wsrep_flow_control_interval      | [ 173, 173 ]                                       |
| wsrep_flow_control_interval_low  | 173                                                |
| wsrep_flow_control_interval_high | 173                                                |
| wsrep_flow_control_status        | OFF                                                |
| wsrep_cert_deps_distance         | 1.000000                                           |
| wsrep_apply_oooe                 | 0.000000                                           |
| wsrep_apply_oool                 | 0.000000                                           |
| wsrep_apply_window               | 1.000000                                           |
| wsrep_commit_oooe                | 0.000000                                           |
| wsrep_commit_oool                | 0.000000                                           |
| wsrep_commit_window              | 1.000000                                           |
| wsrep_local_state                | 4                                                  |
| wsrep_local_state_comment        | Synced                                             |
| wsrep_cert_index_size            | 3                                                  |
| wsrep_cert_bucket_count          | 22                                                 |
| wsrep_gcache_pool_size           | 5544                                               |
| wsrep_causal_reads               | 0                                                  |
| wsrep_cert_interval              | 0.000000                                           |
| wsrep_open_transactions          | 0                                                  |
| wsrep_open_connections           | 0                                                  |
| wsrep_ist_receive_status         |                                                    |
| wsrep_ist_receive_seqno_start    | 0                                                  |
| wsrep_ist_receive_seqno_current  | 0                                                  |
| wsrep_ist_receive_seqno_end      | 0                                                  |
| wsrep_incoming_addresses         | 10.51.21.10:3306,10.51.21.11:3306,10.51.21.12:3306 |
| wsrep_cluster_weight             | 3                                                  |
| wsrep_desync_count               | 0                                                  |
| wsrep_evs_delayed                |                                                    |
| wsrep_evs_evict_list             |                                                    |
| wsrep_evs_repl_latency           | 0/0/0/0/0                                          |
| wsrep_evs_state                  | OPERATIONAL                                        |
| wsrep_gcomm_uuid                 | d75349c6-d233-11e9-be9f-fa530c0c4068               |
| wsrep_cluster_conf_id            | 2                                                  |
| wsrep_cluster_size               | 3                                                  |
| wsrep_cluster_state_uuid         | c09d5a38-d233-11e9-9d3a-2ae4d93e72bc               |
| wsrep_cluster_status             | Primary                                            |
| wsrep_connected                  | ON                                                 |
| wsrep_local_bf_aborts            | 0                                                  |
| wsrep_local_index                | 2                                                  |
| wsrep_provider_name              | Galera                                             |
| wsrep_provider_vendor            | Codership Oy <info@codership.com>                  |
| wsrep_provider_version           | 3.37(rff05089)                                     |
| wsrep_ready                      | ON                                                 |
+----------------------------------+----------------------------------------------------+
71 rows in set (0.01 sec)
```

---
title: Зниження версії etcd з 3.6 до 3.5
weight: 6650
description: Процеси, контрольні списки та примітки щодо зниження версії etcd з 3.6 до 3.5
---

У загальному випадку, зниження версії з etcd v3.6 до v3.5 може бути безперервним, поступовим зниженням версії без простою:

- по черзі зупиняйте процеси etcd v3.6 і замінюйте їх процесами etcd v3.5
- після увімкнення зниження версії нові функції у v3.6 більше не доступні для кластера

Перед тим як [почати зниження версії](#downgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки для зниження версії {#downgrade-checklists}

Основні зміни між v3.6 та v3.5:

#### Відмінності у прапорцях {#difference-in-flags}

Якщо ви використовуєте будь-які з наступних прапорців у ваших конфігураціях v3.6, переконайтеся, що ви видалили, перейменували або змінили стандартне значення при зниженні до v3.5.

{{% alert title="Примітка" color="info" %}}

Різниця базується на версіях v3.6.0 та v.3.5.18. Фактична різниця залежатиме від вашої патч-версії, перевірте за допомогою `diff <(etcd-3.6/bin/etcd -h | grep \\-\\-) <(etcd-3.5/bin/etcd -h | grep \\-\\-)` спочатку.

{{% /alert %}}

```diff
# прапорці, які недоступні у v3.5
-etcd --discovery-token ''
-etcd --discovery-endpoints ''
-etcd --discovery-dial-timeout '2s'
-etcd --discovery-request-timeout '5s'
-etcd --discovery-keepalive-time '2s'
-etcd --discovery-keepalive-timeout '6s'
-etcd --discovery-insecure-transport 'true'
-etcd --discovery-insecure-skip-tls-verify 'false'
-etcd --discovery-cert ''
-etcd --discovery-key ''
-etcd --discovery-cacert ''
-etcd --discovery-user ''
-etcd --discovery-password ''
-etcd --feature-gates
-etcd --log-format

# ті ж самі прапорці з іншими назвами
-etcd --bootstrap-defrag-threshold-megabytes
+etcd --experimental-bootstrap-defrag-threshold-megabytes
-etcd --compaction-batch-limit
+etcd --experimental-compaction-batch-limit
-etcd --compact-hash-check-time
+etcd --experimental-compact-hash-check-time
-etcd --compaction-sleep-interval
+etcd --experimental-compaction-sleep-interval
-etcd --corrupt-check-time
+etcd --experimental-corrupt-check-time
-etcd --enable-distributed-tracing
+etcd --experimental-enable-distributed-tracing
-etcd --distributed-tracing-address
+etcd --experimental-distributed-tracing-address
-etcd --distributed-tracing-instance-id
+etcd --experimental-distributed-tracing-instance-id
-etcd --distributed-tracing-sampling-rate
+etcd --experimental-distributed-tracing-sampling-rate
-etcd --distributed-tracing-service-name
+etcd --experimental-distributed-tracing-service-name
-etcd --downgrade-check-time
+etcd --experimental-downgrade-check-time
-etcd --max-learners
+etcd --experimental-max-learners
-etcd --memory-mlock
+etcd --experimental-memory-mlock
-etcd --peer-skip-client-san-verification
+etcd --experimental-peer-skip-client-san-verification
-etcd --snapshot-catchup-entries
+etcd --experimental-snapshot-catchup-entries
-etcd --warning-apply-duration
+etcd --experimental-warning-apply-duration
-etcd --warning-unary-request-duration
+etcd --experimental-warning-unary-request-duration
-etcd --watch-progress-notify-interval
+etcd --experimental-watch-progress-notify-interval

# еквівалентні прапорці функціональних можливостей v3.6
-etcd --feature-gates=CompactHashCheck=true
+etcd --experimental-compact-hash-check-enabled=true
-etcd --feature-gates=InitialCorruptCheck=true
+etcd --experimental-enable-initial-corrupt-check=true
-etcd --feature-gates=LeaseCheckpoint=true
+etcd --experimental-enable-lease-checkpoint=true
-etcd --feature-gates=LeaseCheckpointPersist=true
+etcd --experimental-enable-lease-checkpoint-persist=true
-etcd --feature-gates=StopGRPCServiceOnDefrag=true
+etcd --experimental-stop-grpc-service-on-defrag=true
-etcd --feature-gates=TxnModeWriteWithSharedBuffer=false
+etcd --experimental-txn-mode-write-with-shared-buffer=false

# ті ж самі прапорці з різними стандартними значеннями
-etcd --snapshot-count=10000
+etcd --snapshot-count=100000
-etcd --v2-deprecation='write-only'
+etcd --v2-deprecation='not-yet'
-etcd --discovery-fallback='exit'
+etcd --discovery-fallback='proxy'

```

#### Відмінності у метриках Prometheus {#difference-in-prometheus-metrics}

```diff
# метрики, які недоступні у v3.5
-etcd_network_known_peers
-etcd_server_feature_enabled
```

### Контрольні списки для зниження версії сервера {#server-downgrade-checklists}

#### Вимоги до зниження версії {#downgrade-requirements}

Щоб забезпечити плавне поступове зниження версії, кластер, що працює, повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед зниженням версії etcd завжди тестуйте сервіси, що залежать від etcd, у тестовому середовищі перед розгортанням зниження версії у виробничому середовищі.

Перед початком [завантажте резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup). Якщо щось піде не так зі зниженням версії, можна використовувати цю резервну копію для [відкату](#rollback) до наявної версії etcd.

Перед початком завантажте останній випуск etcd v3.5.

#### Змішані версії {#mixed-versions}

Під час переходу на нижчу версію кластер etcd підтримує змішані версії вузлів etcd і працює за протоколом найнижчої спільної версії. Кластер вважається таким, що перейшов на нижчу версію, після увімкнення переходу командою `etcdctl downgrade enable 3.5`. Внутрішньо загальна версія кластера встановлюється на цільову версію переходу, яка визначає версію, що повідомляється, та підтримувані функції.

#### Відкат {#rollback}

Перед зниженням версії вашого кластера etcd створіть та [завантажте резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup) вашого кластера etcd. Цей знімок можна використовувати для відновлення кластера до стану до оновлення за потреби. Якщо під час зниження версії виникають проблеми, спочатку визначте та усуньте їхню першопричину.

Якщо зниження версії було запущено після виконання `etcdctl downgrade enabled`, і кластер все ще перебуває у стані змішаних версій, де хоча б один член залишається на v3.6, користувачі можуть скасувати тривалий процес зниження версії за допомогою `etcdctl downgrade cancel` та перезапустити всі знижені члени з оригінальними двійковими файлами v3.6.

Після зниження всіх членів до v3.5 кластер вважається повністю зниженим. Якщо користувачі бажають повернутися до початкової версії після завершення повного зниження, їм слід дотримуватися офіційного [посібника з оновлення](/docs/v3.8/upgrades/upgrade_3_6/), щоб забезпечити цілісність та уникнути пошкодження даних.

### Процедура зниження версії {#downgrade-procedure}

Цей приклад показує, як знизити версію кластера etcd v3.6 з 3 членами, що працює на локальній машині.

#### Крок 1: перевірте вимоги до зниження версії {#step-1-check-downgrade-requirements}

Чи кластер справний та працює на v3.6.x?

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint health
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 2.118638ms
localhost:22379 is healthy: successfully committed proposal: took = 3.631388ms
localhost:32379 is healthy: successfully committed proposal: took = 2.157051ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.6.0-alpha.0","etcdcluster":"3.6.0","storage":"3.6.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.6.0-alpha.0","etcdcluster":"3.6.0","storage":"3.6.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.6.0-alpha.0","etcdcluster":"3.6.0","storage":"3.6.0"}
COMMENT

etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |    VERSION    | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 8211f1d0f64f3269 | 3.6.0-alpha.0 |           3.6.0 |   20 kB |  16 kB |                   20% |   0 B |      true |      false |         2 |         10 |                 10 |        |                          |             false |
| localhost:22379 | 91bc3c398fb3c146 | 3.6.0-alpha.0 |           3.6.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         2 |         10 |                 10 |        |                          |             false |
| localhost:32379 | fd422379fda50e48 | 3.6.0-alpha.0 |           3.6.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         2 |         10 |                 10 |        |                          |             false |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT
```

#### Крок 2: завантажте резервну копію знімка з лідера {#step-2-download-snapshot-backup-from-leader}

[Завантажте резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup), щоб забезпечити шлях для зниження версії у разі виникнення проблем.

#### Крок 3: перевірте цільову версію зниження {#step-3-validate-downgrade-target-version}

Перевірте цільову версію зниження перед увімкненням зниження версії:

- Ми підтримуємо зниження версії лише на одну мінорну версію за раз. Наприклад, зниження з v3.6 до v3.4 не допускається.
- Не переходьте до наступного кроку, поки перевірка не буде успішною.

```bash
etcdctl downgrade validate 3.5
<<COMMENT
Downgrade validate success, cluster version 3.6
COMMENT
```

#### Крок 4: увімкніть зниження версії {#step-4-enable-downgrade}

```bash
etcdctl downgrade enable 3.5
<<COMMENT
Downgrade enable success, cluster version 3.6
COMMENT
```

Після увімкнення зниження версії кластер почне працювати з протоколом v3.5, який є цільовою версією зниження. Крім того, etcd автоматично мігрує схему до цільової версії зниження, що зазвичай відбувається дуже швидко. Переконайтеся, що версія сховища всіх серверів була мігрована до v3.5, перевіривши стан точки доступу перед переходом до наступного кроку.

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |    VERSION    | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 8211f1d0f64f3269 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |      true |      false |         2 |         12 |                 12 |        |                    3.5.0 |              true |
| localhost:22379 | 91bc3c398fb3c146 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         2 |         12 |                 12 |        |                    3.5.0 |              true |
| localhost:32379 | fd422379fda50e48 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         2 |         12 |                 12 |        |                    3.5.0 |              true |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT
```

{{% alert title="Примітка" color="info" %}}

Після увімкнення зниження версії кластер продовжить працювати з протоколом v3.5, навіть якщо всі сервери все ще працюють з двійковим файлом v3.6, доки зниження версії не буде скасовано за допомогою `etcdctl downgrade cancel`.

{{% /alert %}}

#### Крок 5: зупиніть один існуючий сервер etcd {#step-5-stop-one-existing-etcd-server}

Перед зупинкою сервера перевірте, чи є він лідером. Рекомендується знижувати версію лідера останнім.

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |    VERSION    | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 8211f1d0f64f3269 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |      true |      false |         2 |         12 |                 12 |        |                    3.5.0 |              true |
| localhost:22379 | 91bc3c398fb3c146 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         2 |         12 |                 12 |        |                    3.5.0 |              true |
| localhost:32379 | fd422379fda50e48 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         2 |         12 |                 12 |        |                    3.5.0 |              true |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT
```

Якщо сервер, який потрібно зупинити, є лідером, ви можете уникнути деякого простою, перемістивши лідера на інший сервер за допомогою `move-leader` перед зупинкою цього сервера.

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 move-leader 91bc3c398fb3c146

etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |    VERSION    | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 8211f1d0f64f3269 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         3 |         13 |                 13 |        |                    3.5.0 |              true |
| localhost:22379 | 91bc3c398fb3c146 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |      true |      false |         3 |         13 |                 13 |        |                    3.5.0 |              true |
| localhost:32379 | fd422379fda50e48 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         3 |         13 |                 13 |        |                    3.5.0 |              true |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT
```

Коли кожен процес etcd зупиняється, очікувані помилки будуть записані іншими членами кластера. Це нормально, оскільки зʼєднання члена кластера було (тимчасово) розірвано:

```bash
{"level":"warn","ts":"2025-02-28T17:35:43.795069Z","caller":"etcdserver/cluster_util.go:259","msg":"failed to reach the peer URL","address":"http://127.0.0.1:12380/version","remote-member-id":"8211f1d0f64f3269","error":"Get \"http://127.0.0.1:12380/version\": dial tcp 127.0.0.1:12380: connect: connection refused"}
{"level":"warn","ts":"2025-02-28T17:35:43.795149Z","caller":"etcdserver/cluster_util.go:160","msg":"failed to get version","remote-member-id":"8211f1d0f64f3269","error":"Get \"http://127.0.0.1:12380/version\": dial tcp 127.0.0.1:12380: connect: connection refused"}
{"level":"warn","ts":"2025-02-28T17:35:44.368651Z","caller":"rafthttp/probing_status.go:68","msg":"prober detected unhealthy status","round-tripper-name":"ROUND_TRIPPER_SNAPSHOT","remote-peer-id":"8211f1d0f64f3269","rtt":"483.01µs","error":"dial tcp 127.0.0.1:12380: connect: connection refused"}
{"level":"warn","ts":"2025-02-28T17:35:44.368726Z","caller":"rafthttp/probing_status.go:68","msg":"prober detected unhealthy status","round-tripper-name":"ROUND_TRIPPER_RAFT_MESSAGE","remote-peer-id":"8211f1d0f64f3269","rtt":"735.659µs","error":"dial tcp 127.0.0.1:12380: connect: connection refused"}
```

#### Крок 6: перезапустіть сервер etcd з тією ж конфігурацією (за винятком прапорців, які видалено або замінено у v3.5) {#step-6-restart-the-etcd-server-with-same-configuration-minus-the-flags-that-are-removed-or-replaced-in-v35}

Перезапустіть сервер etcd з тією ж конфігурацією, але з новим двійковим файлом etcd.

```diff
-etcd-3.6/bin --name s1 \
+etcd-3.5/bin --name s1 \
  --data-dir /tmp/etcd/s1 \
  --listen-client-urls http://localhost:2379 \
  --advertise-client-urls http://localhost:2379 \
  --listen-peer-urls http://localhost:2380 \
  --initial-advertise-peer-urls http://localhost:2380 \
  --initial-cluster s1=http://localhost:2380,s2=http://localhost:22380,s3=http://localhost:32380 \
  --initial-cluster-token tkn \
  --initial-cluster-state existing
```

Переконайтеся, що кожен член, а потім і весь кластер, стає справним з новим двійковим файлом etcd v3.5:

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |    VERSION    | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 8211f1d0f64f3269 |        3.5.18 |                 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         3 |         14 |                 14 |        |                          |             false |
| localhost:22379 | 91bc3c398fb3c146 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |      true |      false |         3 |         14 |                 14 |        |                    3.5.0 |              true |
| localhost:32379 | fd422379fda50e48 | 3.6.0-alpha.0 |           3.5.0 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         3 |         14 |                 14 |        |                    3.5.0 |              true |
+-----------------+------------------+---------------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT

etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:22379 is healthy: successfully committed proposal: took = 4.650967ms
localhost:2379 is healthy: successfully committed proposal: took = 4.634377ms
localhost:32379 is healthy: successfully committed proposal: took = 5.047777ms
COMMENT
```

{{% alert title="Примітка" color="info" %}}

Ви побачите, що `DOWNGRADE ENABLED` має значення false для сервера v3.5, оскільки інформація про зниження версії не реалізована в endpoint status v3.5; зниження версії все ще увімкнено для кластера на цьому етапі.

{{% /alert %}}

#### Крок 7: повторіть *крок 5* та *крок 6* для решти членів {#step-7-repeat-step-5-and-step-6-for-rest-of-the-members}

Коли всі члени знижені, перевірте стан здоровʼя та версію кластера і переконайтеся, що мінорна версія всіх членів є v3.5, а версія сховища — порожня:

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        | VERSION | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+---------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 8211f1d0f64f3269 |  3.5.18 |                 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         3 |         26 |                 26 |        |                          |             false |
| localhost:22379 | 91bc3c398fb3c146 |  3.5.18 |                 |   20 kB |  16 kB |                   20% |   0 B |      true |      false |         3 |         26 |                 26 |        |                          |             false |
| localhost:32379 | fd422379fda50e48 |  3.5.18 |                 |   20 kB |  16 kB |                   20% |   0 B |     false |      false |         3 |         26 |                 26 |        |                          |             false |
+-----------------+------------------+---------+-----------------+---------+--------+-----------------------+-------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT

etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:22379 is healthy: successfully committed proposal: took = 4.650967ms
localhost:2379 is healthy: successfully committed proposal: took = 4.634377ms
localhost:32379 is healthy: successfully committed proposal: took = 5.047777ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.5.18","etcdcluster":"3.5.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.5.18","etcdcluster":"3.5.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.5.18","etcdcluster":"3.5.0"}
COMMENT
```

У журналі лідера ви повинні побачити повідомлення, подібне до наступного:

```bash
{"level":"info","ts":"2025-02-28T17:59:50.019862Z","caller":"etcdserver/server.go:2749","msg":"the cluster has been downgraded","cluster-version":"3.5.0"}
```

---
title: Зниження версії etcd з 3.5 до 3.4
weight: 6650
description: Процеси, контрольні списки та примітки щодо зниження версії etcd з 3.5 до 3.4
---

У загальному випадку, зниження версії з etcd 3.5 до 3.4 може бути безперервним, поступовим зниженням версії:

- по черзі зупиняйте процеси etcd 3.5 і замінюйте їх процесами etcd 3.4
- після запуску будь-яких процесів 3.4 нові функції в 3.5 більше не доступні для кластера

Перед тим як [почати зниження версії](#downgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки для зниження версії {#downgrade-checklists}

**ПРИМІТКА:** Якщо у вашому кластері увімкнено автентифікацію, поступове зниження версії з 3.5 не підтримується, оскільки 3.5 [змінює формат записів WAL, повʼязаних з автентифікацією](https://github.com/etcd-io/etcd/pull/11943). Ви можете дотримуватися [інструкцій з автентифікації](../../op-guide/authentication/rbac/), щоб вимкнути автентифікацію та видалити всіх користувачів спочатку.

Основні зміни між 3.5 та 3.4:

#### Відмінності у прапорцях {#difference-in-flags}

Якщо ви використовуєте будь-які з наступних прапорців у ваших конфігураціях 3.5, переконайтеся, що ви видалили, перейменували або змінили стандартні значення при зниженні до 3.4.

**ПРИМІТКА** Різниця базується на версіях 3.5.14 та 3.4.33. Фактична різниця залежатиме від вашої патч-версії, перевірте за допомогою `diff <(etcd-3.5/bin/etcd -h | grep \\-\\-) <(etcd-3.4/bin/etcd -h | grep \\-\\-)` спочатку.

```diff
# прапорці, які недоступні в 3.4
-etcd --socket-reuse-port
-etcd --socket-reuse-address
-etcd --raft-read-timeout
-etcd --raft-write-timeout
-etcd --v2-deprecation
-etcd --client-cert-file
-etcd --client-key-file
-etcd --peer-client-cert-file
-etcd --peer-client-key-file
-etcd --self-signed-cert-validity
-etcd --enable-log-rotation --log-rotation-config-json=some.json
-etcd --experimental-enable-distributed-tracing --experimental-distributed-tracing-address='localhost:4317' --experimental-distributed-tracing-service-name='etcd' --experimental-distributed-tracing-instance-id='' --experimental-distributed-tracing-sampling-rate='0'
-etcd --experimental-compact-hash-check-enabled --experimental-compact-hash-check-time='1m'
-etcd --experimental-downgrade-check-time
-etcd --experimental-memory-mlock
-etcd --experimental-txn-mode-write-with-shared-buffer
-etcd --experimental-bootstrap-defrag-threshold-megabytes
-etcd --experimental-stop-grpc-service-on-defrag

# той самий прапорець з іншими назвами
-etcd --backend-bbolt-freelist-type=map
+etcd --experimental-backend-bbolt-freelist-type=array

# той самий прапорець з іншими стандартними значеннями
-etcd --pre-vote=true
+etcd --pre-vote=false

-etcd --logger=zap
+etcd --logger=capnslog
```

#### `etcd --logger zap` {#etcd-logger-zap}

3.4 стандартно використовує `--logger=capnslog`, тоді як 3.5 використовує `--logger=zap`.

Якщо ви хочете продовжувати використовувати `zap`, його потрібно вказати явно.

```diff
+etcd --logger=zap --log-outputs=stderr

+# щоб записувати логи одночасно в stderr та a.log файл
+etcd --logger=zap --log-outputs=stderr,a.log
```

#### Відмінності у метриках Prometheus {#difference-in-prometheus-metrics}

```diff
# метрики, які недоступні в 3.4
-etcd_debugging_mvcc_db_compaction_last
```

### Контрольні списки для зниження версії сервера {#server-downgrade-checklists}

#### Вимоги до зниження версії {#downgrade-requirements}

Щоб забезпечити плавне поступове зниження версії, кластер, що працює, повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

Версія 3.4, до якої потрібно знизити, повинна бути >= 3.4.32.

#### Підготовка {#preparation}

Перед зниженням версії etcd завжди тестуйте сервіси, що залежать від etcd, у тестовому середовищі перед розгортанням зниження версії у виробничому середовищі.

Перед початком [завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup). Якщо щось піде не так зі зниженням версії, можна використовувати цю резервну копію для [відкату](#rollback) до поточної версії etcd. Зверніть увагу, що команда `snapshot` резервує лише дані v3. Для даних v2 див. [резервне копіювання v2 сховища](/docs/v2.3/admin_guide#backing-up-the-datastore).

Перед початком завантажте останній випуск etcd 3.4 і переконайтеся, що його версія >= 3.4.32.

#### Змішані версії {#mixed-versions}

Під час зниження версії кластер etcd підтримує змішані версії членів etcd і працює з протоколом найнижчої загальної версії. Кластер вважається зниженим, як тільки будь-який з його членів знижено до версії 3.4. Внутрішньо члени etcd домовляються між собою, щоб визначити загальну версію кластера, яка контролює повідомлену версію та підтримувані функції.

#### Обмеження {#limitations}

Примітка: Якщо кластер містить лише дані v3 і не містить даних v2, це обмеження не застосовується.

Якщо кластер обслуговує набір даних v2 розміром понад 50 МБ, кожен новий знижений член може зайняти до двох хвилин, щоб наздогнати наявний кластер. Перевірте розмір останнього знімка, щоб оцінити загальний розмір даних. Іншими словами, найкраще почекати 2 хвилини між зниженням кожного члена.

Для набагато більшого загального розміру даних, 100 МБ або більше, цей одноразовий процес може зайняти ще більше часу. Адміністратори дуже великих кластерів etcd такого масштабу можуть звернутися до [команди etcd][etcd-contact] перед зниженням версії, і ми будемо раді надати поради щодо процедури.

#### Відкат {#rollback}

Якщо будь-який член був знижений до 3.4, версія кластера буде знижена до 3.4, і операції будуть сумісні з "3.4". Вам потрібно буде дотримуватися інструкцій [Оновлення etcd з 3.4 до 3.5](../../upgrades/upgrade_3_5/), щоб відкотитися.

Будь ласка, [завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup), щоб зробити можливим зниження версії кластера навіть після його повного зниження.

### Процедура зниження версії {#downgrade-procedure}

Цей приклад показує, як знизити версію кластера etcd з 3 членами 3.5, що працює на локальній машині.

#### Крок 1: перевірте вимоги до зниження версії {#step-1-check-downgrade-requirements}

Чи кластер справний та працює на 3.5.x?

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint health
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 2.118638ms
localhost:22379 is healthy: successfully committed proposal: took = 3.631388ms
localhost:32379 is healthy: successfully committed proposal: took = 2.157051ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.5.0","etcdcluster":"3.5.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.5.0","etcdcluster":"3.5.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.5.0","etcdcluster":"3.5.0"}
COMMENT
```

#### Крок 2: завантажте резервну копію знімка з лідера {#step-2-download-snapshot-backup-from-leader}

[Завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup), щоб забезпечити шлях для зниження версії у разі виникнення проблем.

#### Крок 3: зупиніть один існуючий сервер etcd {#step-3-stop-one-existing-etcd-server}

Перед зупинкою сервера перевірте, чи є він лідером

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT     |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|  localhost:2379 | 8211f1d0f64f3269 |  3.5.13 |   20 kB |      true |      false |         2 |          9 |                  9 |        |
| localhost:22379 | 91bc3c398fb3c146 |  3.5.13 |   20 kB |     false |      false |         2 |          9 |                  9 |        |
| localhost:32379 | fd422379fda50e48 |  3.5.13 |   20 kB |     false |      false |         2 |          9 |                  9 |        |
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
COMMENT
```

Якщо сервер, який потрібно зупинити, є лідером, ви можете уникнути деякого простою, перемістивши лідера на інший сервер перед зупинкою цього сервера.

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 move-leader 91bc3c398fb3c146

etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT     |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|  localhost:2379 | 8211f1d0f64f3269 |  3.5.13 |   20 kB |     false |      false |         3 |         11 |                 11 |        |
| localhost:22379 | 91bc3c398fb3c146 |  3.5.13 |   20 kB |      true |      false |         3 |         11 |                 11 |        |
| localhost:32379 | fd422379fda50e48 |  3.5.13 |   20 kB |     false |      false |         3 |         11 |                 11 |        |
+-----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
COMMENT
```

Коли кожен процес etcd зупиняється, очікувані помилки будуть записані іншими членами кластера. Це нормально, оскільки зʼєднання члена кластера було (тимчасово) розірвано:

```bash
{"level":"info","ts":"2024-05-14T20:25:47.051124Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"91bc3c398fb3c146 became leader at term 3"}
{"level":"info","ts":"2024-05-14T20:25:47.051139Z","logger":"raft","caller":"etcdserver/zap_raft.go:77","msg":"raft.node: 91bc3c398fb3c146 elected leader 91bc3c398fb3c146 at term 3"}

^C{"level":"warn","ts":"2024-05-14T20:27:09.094119Z","caller":"rafthttp/stream.go:421","msg":"lost TCP streaming connection with remote peer","stream-reader-type":"stream MsgApp v2","local-member-id":"91bc3c398fb3c146","remote-peer-id":"8211f1d0f64f3269","error":"EOF"}
{"level":"warn","ts":"2024-05-14T20:27:09.09427Z","caller":"rafthttp/stream.go:421","msg":"lost TCP streaming connection with remote peer","stream-reader-type":"stream Message","local-member-id":"91bc3c398fb3c146","remote-peer-id":"8211f1d0f64f3269","error":"EOF"}
{"level":"warn","ts":"2024-05-14T20:27:09.095535Z","caller":"rafthttp/peer_status.go:66","msg":"peer became inactive (message send to peer failed)","peer-id":"8211f1d0f64f3269","error":"failed to dial 8211f1d0f64f3269 on stream MsgApp v2 (peer 8211f1d0f64f3269 failed to find local node 91bc3c398fb3c146)"}
{"level":"warn","ts":"2024-05-14T20:27:09.43915Z","caller":"rafthttp/stream.go:223","msg":"lost TCP streaming connection with remote peer","stream-writer-type":"stream Message","local-member-id":"91bc3c398fb3c146","remote-peer-id":"8211f1d0f64f3269"}
{"level":"warn","ts":"2024-05-14T20:27:11.085646Z","caller":"etcdserver/cluster_util.go:294","msg":"failed to reach the peer URL","address":"http://127.0.0.1:12380/version","remote-member-id":"8211f1d0f64f3269","error":"Get \"http://127.0.0.1:12380/version\": dial tcp 127.0.0.1:12380: connect: connection refused"}
{"level":"warn","ts":"2024-05-14T20:27:11.085718Z","caller":"etcdserver/cluster_util.go:158","msg":"failed to get version","remote-member-id":"8211f1d0f64f3269","error":"Get \"http://127.0.0.1:12380/version\": dial tcp 127.0.0.1:12380: connect: connection refused"}
{"level":"warn","ts":"2024-05-14T20:27:13.557385Z","caller":"rafthttp/probing_status.go:68","msg":"prober detected unhealthy status","round-tripper-name":"ROUND_TRIPPER_SNAPSHOT","remote-peer-id":"8211f1d0f64f3269","rtt":"416.079µs","error":"dial tcp 127.0.0.1:12380: connect: connection refused"}
```

#### Крок 4: перезапустіть сервер etcd з тією ж конфігурацією + `--next-cluster-version-compatible` {#step-4-restart-the-etcd-server-with-same-configuration-next-cluster-version-compatible}

Перезапустіть сервер etcd з тією ж конфігурацією, але з новим двійковим файлом etcd і `--next-cluster-version-compatible`.

```diff
-etcd-old --name s1 \
+etcd-new --name s1 \
  --data-dir /tmp/etcd/s1 \
  --listen-client-urls http://localhost:2379 \
  --advertise-client-urls http://localhost:2379 \
  --listen-peer-urls http://localhost:2380 \
  --initial-advertise-peer-urls http://localhost:2380 \
  --initial-cluster s1=http://localhost:2380,s2=http://localhost:22380,s3=http://localhost:32380 \
  --initial-cluster-token tkn \
  --initial-cluster-state existing
  --next-cluster-version-compatible
```

Новий etcd 3.4 опублікує свою інформацію в кластері. На цьому етапі кластер почне працювати за протоколом 3.4, який є найнижчою загальною версією.

> `{"level":"info","ts":"2024-05-13T21:05:43.981445Z","caller":"membership/cluster.go:561","msg":"set initial cluster version","cluster-id":"ef37ad9dc622a7c4","local-member-id":"8211f1d0f64f3269","cluster-version":"3.0"}`

> `{"level":"info","ts":"2024-05-13T21:05:43.982188Z","caller":"api/capability.go:77","msg":"enabled capabilities for version","cluster-version":"3.0"}`

> `{"level":"info","ts":"2024-05-13T21:05:43.982312Z","caller":"membership/cluster.go:549","msg":"updated cluster version","cluster-id":"ef37ad9dc622a7c4","local-member-id":"8211f1d0f64f3269","from":"3.0","from":"3.5"}`

> `{"level":"info","ts":"2024-05-13T21:05:43.982376Z","caller":"api/capability.go:77","msg":"enabled capabilities for version","cluster-version":"3.5"}`

> `{"level":"info","ts":"2024-05-13T21:05:44.000672Z","caller":"etcdserver/server.go:2152","msg":"published local member to cluster through raft","local-member-id":"8211f1d0f64f3269","local-member-attributes":"{Name:infra1 ClientURLs:[http://127.0.0.1:2379]}","request-path":"/0/members/8211f1d0f64f3269/attributes","cluster-id":"ef37ad9dc622a7c4","publish-timeout":"7s"}`

> `{"level":"info","ts":"2024-05-13T21:05:46.452631Z","caller":"membership/cluster.go:549","msg":"updated cluster version","cluster-id":"ef37ad9dc622a7c4","local-member-id":"8211f1d0f64f3269","from":"3.5","from":"3.4"}`

Переконайтеся, що кожен член, а потім весь кластер, стає справним з новим двійковим файлом etcd 3.4:

```bash
etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:32379 is healthy: successfully committed proposal: took = 2.337471ms
localhost:22379 is healthy: successfully committed proposal: took = 1.130717ms
localhost:2379 is healthy: successfully committed proposal: took = 2.124843ms
COMMENT
```

Не знижені члени будуть записувати інформацію, як показано нижче

```log
{"level":"info","ts":"2024-05-13T21:05:46.450764Z","caller":"etcdserver/server.go:2633","msg":"updating cluster version using v2 API","from":"3.5","to":"3.4"}
{"level":"info","ts":"2024-05-13T21:05:46.452419Z","caller":"membership/cluster.go:576","msg":"updated cluster version","cluster-id":"ef37ad9dc622a7c4","local-member-id":"91bc3c398fb3c146","from":"3.5","to":"3.4"}
{"level":"info","ts":"2024-05-13T21:05:46.452547Z","caller":"etcdserver/server.go:2652","msg":"cluster version is updated","cluster-version":"3.4"}
```

#### Крок 5: повторіть *крок 3* та *крок 4* для решти членів {#step-5-repeat-step-3-and-step-4-for-rest-of-the-members}

Коли всі члени знижені, перевірте стан здоровʼя та версію кластера:

```bash
endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 492.834µs
localhost:22379 is healthy: successfully committed proposal: took = 1.015025ms
localhost:32379 is healthy: successfully committed proposal: took = 1.853077ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.4.32","etcdcluster":"3.4.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.4.32","etcdcluster":"3.4.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.4.32","etcdcluster":"3.4.0"}
COMMENT
```

[etcd-contact]: https://groups.google.com/g/etcd-dev

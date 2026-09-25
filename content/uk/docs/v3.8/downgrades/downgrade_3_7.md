---
title: Зниження версії etcd з 3.7 до 3.6
weight: 6600
description: Процеси, контрольні списки та примітки щодо зниження версії etcd з 3.7 до 3.6
---

У загальному випадку, зниження версії з etcd v3.7 до v3.6 може бути безперервним, поступовим зниженням версії без простою:

- по черзі зупиняйте процеси etcd v3.7 і замінюйте їх процесами etcd v3.6
- після увімкнення зниження версії нові функції у v3.7 більше не доступні для кластера

Перед тим як [почати зниження версії](#downgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки для зниження версії {#downgrade-checklists}

Основні зміни між v3.7 та v3.6:

#### Відмінності у прапорцях {#difference-in-flags}

v3.7 не вводить жодних нових прапорців, тому процес v3.6 приймає кожен прапорець конфігурації v3.7, і жодних змін конфігурації не потрібно при зниженні версії.

{{% alert title="Примітка" color="info" %}}
Різниця базується на версіях v3.7.0-rc.0 та v3.6.13. Фактична різниця залежатиме від вашої патч-версії, перевірте за допомогою `diff <(etcd-3.7/bin/etcd -h | grep \\-\\-) <(etcd-3.6/bin/etcd -h | grep \\-\\-)` спочатку.
{{% /alert %}}

Застарілі прапорці `--experimental-*`, які були видалені у v3.7, все ще існують у v3.6, але не додавайте їх повторно після зниження версії; використовуйте їхні неекспериментальні еквіваленти або записи `--feature-gates`, які працюють в обох версіях.

#### Відмінності у метриках Prometheus {#difference-in-prometheus-metrics}

```diff
# метрики відсутні в v3.6
-etcd_server_request_duration_seconds
-etcd_debugging_server_watch_send_loop_control_stream_duration_seconds
-etcd_debugging_server_watch_send_loop_progress_duration_seconds
-etcd_debugging_server_watch_send_loop_watch_stream_duration_seconds
-etcd_debugging_server_watch_send_loop_watch_stream_duration_per_event_seconds
```

### Контрольні списки для зниження версії сервера {#server-downgrade-checklists}

#### Вимоги до зниження версії {#downgrade-requirements}

Щоб забезпечити плавне поступове зниження версії, кластер, що працює, повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед зниженням версії etcd завжди тестуйте сервіси, що залежать від etcd, у тестовому середовищі перед розгортанням зниження версії у виробничому середовищі.

Перед початком [завантажте собі резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup). Якщо щось піде не так зі зниженням версії, можна використовувати цю резервну копію для [відкату](#rollback) до наявної версії etcd.

Перед початком завантажте останній випуск etcd v3.6.

#### Змішані версії {#mixed-versions}

Під час переходу на нижчу версію кластер etcd підтримує змішані версії вузлів etcd і працює за протоколом найнижчої спільної версії. Кластер вважається таким, що перейшов на нижчу версію, після увімкнення переходу командою `etcdctl downgrade enable 3.6`. Внутрішньо загальна версія кластера встановлюється на цільову версію переходу, яка визначає версію, що повідомляється, та підтримувані функції.

#### Відкат {#rollback}

Перед зниженням версії вашого кластера etcd створіть та [завантажте собі резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup) вашого кластера etcd. Цей знімок можна використовувати для відновлення кластера до стану до зниження версії за потреби. Якщо під час зниження версії виникають проблеми, спочатку визначте та усуньте їхню першопричину.

Якщо зниження версії було запущено після виконання `etcdctl downgrade enable`, і кластер все ще перебуває у стані змішаних версій, де хоча б один член залишається на v3.7, користувачі можуть скасувати тривалий процес зниження версії за допомогою `etcdctl downgrade cancel` та перезапустити всіx членів зі зниженою версією з оригінальними двійковими файлами v3.7.

Після того як усі вузли кластера будуть переведені на v3.6, кластер вважатиметься повністю переведеним на попередню версію. Якщо після завершення повного переходу на попередню версію користувачі бажають повернутися до початкової версії, їм слід дотримуватися офіційного [посібника з оновлення](/docs/v3.8/upgrades/upgrade_3_7/), щоб забезпечити узгодженість та уникнути пошкодження даних.

### Процедура зниження версії {#downgrade-procedure}

Цей приклад показує, як знизити версію кластера etcd v3.7 з 3 членами, що працює на локальній машині. Вивід нижче отримано з реального запуску etcd v3.7.0-rc.0 та etcd v3.6.13 на одному хості з трьома портами loopback, на кластері, який було оновлено з v3.6.13 незадовго до цього.

#### Крок 1: перевірте вимоги до зниження версії {#step-1-check-downgrade-requirements}

Чи кластер справний та працює на v3.7.x?

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint health
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 1.052416ms
localhost:32379 is healthy: successfully committed proposal: took = 1.11625ms
localhost:22379 is healthy: successfully committed proposal: took = 1.114291ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.7.0-rc.0","etcdcluster":"3.7.0","storage":"3.7.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.7.0-rc.0","etcdcluster":"3.7.0","storage":"3.7.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.7.0-rc.0","etcdcluster":"3.7.0","storage":"3.7.0"}
COMMENT

etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |  VERSION   | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA  | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 7339c4e5e833c029 | 3.7.0-rc.0 |           3.7.0 |   98 kB |  98 kB |                    0% | 2.1 GB |      true |      false |         5 |         20 |                 20 |        |                          |             false |
| localhost:22379 | 729934363faa4a24 | 3.7.0-rc.0 |           3.7.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         5 |         20 |                 20 |        |                          |             false |
| localhost:32379 |  b548c2511513015 | 3.7.0-rc.0 |           3.7.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         5 |         20 |                 20 |        |                          |             false |
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT
```

#### Крок 2: завантажте резервну копію знімка з лідера {#step-2-download-snapshot-backup-from-leader}

[Завантажте резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup), щоб забезпечити шлях для зниження версії у разі виникнення проблем:

```bash
etcdctl --endpoints=localhost:2379 snapshot save backup.db
<<COMMENT
{"level":"info","ts":"2026-07-02T06:48:11.091982+0300","caller":"snapshot/v3_snapshot.go:83","msg":"created temporary db file","path":"backup.db.part"}
{"level":"info","ts":"2026-07-02T06:48:11.092253+0300","logger":"client","caller":"v3/maintenance.go:236","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":"2026-07-02T06:48:11.099884+0300","caller":"snapshot/v3_snapshot.go:96","msg":"fetching snapshot","endpoint":"localhost:2379"}
{"level":"info","ts":"2026-07-02T06:48:11.100394+0300","logger":"client","caller":"v3/maintenance.go:302","msg":"completed snapshot read; closing"}
{"level":"info","ts":"2026-07-02T06:48:11.103116+0300","caller":"snapshot/v3_snapshot.go:111","msg":"fetched snapshot","endpoint":"localhost:2379","size":"98 kB","took":"10.9815ms","etcd-version":"3.7.0"}
{"level":"info","ts":"2026-07-02T06:48:11.103296+0300","caller":"snapshot/v3_snapshot.go:121","msg":"saved","path":"backup.db"}
Snapshot saved at backup.db
Server version 3.7.0
COMMENT
```

#### Крок 3: перевірте цільову версію зниження {#step-3-validate-downgrade-target-version}

Перевірте цільову версію зниження перед увімкненням зниження версії:

- Ми підтримуємо зниження версії лише на одну мінорну версію за раз. Наприклад, зниження з v3.7 до v3.5 не допускається.
- Не переходьте до наступного кроку, поки перевірка не буде успішною.

```bash
etcdctl downgrade validate 3.6
<<COMMENT
Downgrade validate success, cluster version 3.7
COMMENT
```

#### Крок 4: увімкніть зниження версії {#step-4-enable-downgrade}

```bash
etcdctl downgrade enable 3.6
<<COMMENT
Downgrade enable success, cluster version 3.7
COMMENT
```

Після увімкнення зниження версії кластер почне працювати з протоколом v3.6, який є цільовою версією зниження. Крім того, etcd автоматично мігрує схему до цільової версії зниження, що зазвичай відбувається дуже швидко. Переконайтеся, що версія сховища всіх серверів була мігрована до v3.6, перевіривши статус точок доступу перед переходом до наступного кроку.

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |  VERSION   | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA  | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 7339c4e5e833c029 | 3.7.0-rc.0 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |      true |      false |         5 |         22 |                 22 |        |                    3.6.0 |              true |
| localhost:22379 | 729934363faa4a24 | 3.7.0-rc.0 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         5 |         22 |                 22 |        |                    3.6.0 |              true |
| localhost:32379 |  b548c2511513015 | 3.7.0-rc.0 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         5 |         22 |                 22 |        |                    3.6.0 |              true |
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT
```

{{% alert title="Примітка" color="info" %}}
Після увімкнення зниження версії кластер продовжить працювати з протоколом v3.6, навіть якщо всі сервери все ще працюють з двійковим файлом v3.7, доки зниження версії не буде скасовано за допомогою `etcdctl downgrade cancel`.
{{% /alert %}}

#### Крок 5: зупиніть один наявний сервер etcd {#step-5-stop-one-existing-etcd-server}

Перед зупинкою сервера перевірте, чи є він лідером. Рекомендується знижувати версію лідера останнім. Якщо сервер, який потрібно зупинити, є лідером, ви можете уникнути деякого простою, перемістивши лідера на інший сервер за допомогою `move-leader` перед зупинкою цього сервера.

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 move-leader 729934363faa4a24
<<COMMENT
Leadership transferred from 7339c4e5e833c029 to 729934363faa4a24
COMMENT
```

Коли кожен процес etcd зупиняється, очікувані помилки будуть записані іншими членами кластера. Це нормально, оскільки зʼєднання члена кластера було (тимчасово) розірвано:

```bash
{"level":"warn","ts":"2026-07-02T06:48:14.518460+0300","caller":"rafthttp/stream.go:227","msg":"lost TCP streaming connection with remote peer","stream-writer-type":"stream Message","local-member-id":"7339c4e5e833c029","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":"2026-07-02T06:48:15.913169+0300","caller":"etcdserver/cluster_util.go:261","msg":"failed to reach the peer URL","address":"http://localhost:22380/version","remote-member-id":"729934363faa4a24","error":"Get \"http://localhost:22380/version\": dial tcp [::1]:22380: connect: connection refused"}
{"level":"warn","ts":"2026-07-02T06:48:15.913364+0300","caller":"etcdserver/cluster_util.go:162","msg":"failed to get version","remote-member-id":"729934363faa4a24","error":"Get \"http://localhost:22380/version\": dial tcp [::1]:22380: connect: connection refused"}
{"level":"warn","ts":"2026-07-02T06:48:16.856521+0300","caller":"version/monitor.go:212","msg":"remotes server has mismatching etcd version","remote-member-id":"b548c2511513015","current-server-version":"3.7.0","target-version":"3.6.0"}
```

#### Крок 6: перезапустіть сервер etcd з тією ж конфігурацією {#step-6-restart-the-etcd-server-with-same-configuration}

Перезапустіть сервер etcd з тією ж конфігурацією, але з двійковим файлом etcd v3.6.

```diff
-etcd-3.7/bin/etcd --name s2 \
+etcd-3.6/bin/etcd --name s2 \
  --data-dir /tmp/etcd/s2 \
  --listen-client-urls http://localhost:22379 \
  --advertise-client-urls http://localhost:22379 \
  --listen-peer-urls http://localhost:22380 \
  --initial-advertise-peer-urls http://localhost:22380 \
  --initial-cluster s1=http://localhost:2380,s2=http://localhost:22380,s3=http://localhost:32380 \
  --initial-cluster-token tkn \
  --initial-cluster-state existing
```

Переконайтеся, що кожен член, а потім і весь кластер, стає справним з двійковим файлом etcd v3.6:

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        |  VERSION   | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA  | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 7339c4e5e833c029 | 3.7.0-rc.0 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |      true |      false |         5 |         23 |                 23 |        |                    3.6.0 |              true |
| localhost:22379 | 729934363faa4a24 |     3.6.13 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         5 |         23 |                 23 |        |                    3.6.0 |              true |
| localhost:32379 |  b548c2511513015 | 3.7.0-rc.0 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         5 |         23 |                 23 |        |                    3.6.0 |              true |
+-----------------+------------------+------------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT

etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 939.625µs
localhost:32379 is healthy: successfully committed proposal: took = 981.459µs
localhost:22379 is healthy: successfully committed proposal: took = 1.11075ms
COMMENT
```

{{% alert title="Примітка" color="info" %}}
На відміну від v3.5, endpoint status у v3.6 повідомляє інформацію про зниження версії, тому знижені члени продовжують показувати `DOWNGRADE ENABLED` як true та свою версію сховища, доки зниження версії не завершиться.
{{% /alert %}}

#### Крок 7: повторіть *крок 5* та *крок 6* для решти членів {#step-7-repeat-step-5-and-step-6-for-rest-of-the-members}

Коли всі члени знижені, зниження версії автоматично завершується, і `DOWNGRADE ENABLED` скидається до false. Перевірте стан справності та статус кластера і переконайтеся, що мінорна версія всіх членів та версія сховища є v3.6:

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w=table
<<COMMENT
+-----------------+------------------+---------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|    ENDPOINT     |        ID        | VERSION | STORAGE VERSION | DB SIZE | IN USE | PERCENTAGE NOT IN USE | QUOTA  | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS | DOWNGRADE TARGET VERSION | DOWNGRADE ENABLED |
+-----------------+------------------+---------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
|  localhost:2379 | 7339c4e5e833c029 |  3.6.13 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         6 |         30 |                 30 |        |                          |             false |
| localhost:22379 | 729934363faa4a24 |  3.6.13 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |      true |      false |         6 |         30 |                 30 |        |                          |             false |
| localhost:32379 |  b548c2511513015 |  3.6.13 |           3.6.0 |   98 kB |  98 kB |                    0% | 2.1 GB |     false |      false |         6 |         30 |                 30 |        |                          |             false |
+-----------------+------------------+---------+-----------------+---------+--------+-----------------------+--------+-----------+------------+-----------+------------+--------------------+--------+--------------------------+-------------------+
COMMENT

etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:22379 is healthy: successfully committed proposal: took = 5.176958ms
localhost:32379 is healthy: successfully committed proposal: took = 5.177875ms
localhost:2379 is healthy: successfully committed proposal: took = 5.191625ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.6.13","etcdcluster":"3.6.0","storage":"3.6.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.6.13","etcdcluster":"3.6.0","storage":"3.6.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.6.13","etcdcluster":"3.6.0","storage":"3.6.0"}
COMMENT
```

У журналі лідера ви повинні побачити повідомлення, подібне до наступного:

```bash
{"level":"info","ts":"2026-07-02T06:48:32.312205+0300","caller":"version/monitor.go:143","msg":"the cluster has been downgraded","cluster-version":"3.6.0"}
```

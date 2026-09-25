---
title: Оновлення etcd з 3.6 до 3.7
weight: 6700
description: Процеси, контрольні списки та примітки щодо оновлення etcd з 3.6 до 3.7
---

У загальному випадку, оновлення з etcd v3.6 до v3.7 може бути безперервним, поступовим оновленням без простою:

- по черзі зупиняйте процеси etcd v3.6 і замінюйте їх процесами etcd v3.7
- після запуску всіх процесів v3.7 нові функції у v3.7 стають доступними для кластера

Перед тим як [почати оновлення](#upgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки оновлення {#upgrade-checklists}

#### Оновлення 3.6 {#update-36}

{{% alert title="Important" color="warning" %}}
Перед оновленням до 3.7 переконайтеся, що всі ваші учасники 3.6 оновлено до 3.6.11 або пізнішої версії. Раніші патч-випуски 3.6 можуть бути несумісними з поступовим оновленням до 3.7.
{{% /alert %}}

#### Сховище V2 {#v2-store}

Сховище v2 було повністю видалено у v3.7. v2 HTTP API (`--enable-v2`), емуляція v2-на-v3 (`--experimental-enable-v2v3`), служба виявлення v2, пакет `client/v2` та завантаження файлів знімків v2 — усе це видалено. Дивіться посилання на зміни, що порушують сумісність, у [CHANGELOG-3.7](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.7.md).

Якщо ви оновлюєтеся з кластера 3.6, ці прапорці вже відсутні, і жодних дій не потрібно. Якщо ви переходите зі старішого випуску з кастомними даними v2, дотримуйтесь [посібника з міграції v2](/docs/v3.4/op-guide/v2-migration/) перед оновленням.

#### Рефакторинг Go {#go-refactoring}

v3.7 містить значний внутрішній рефакторинг, який не впливає на звичайних користувачів, що оновлюються, але про який варто знати при оновленні кастомних інтеграцій:

- Міграція з `gogo/protobuf` на стандартний `google.golang.org/protobuf` (відстежується у [#14533](https://github.com/etcd-io/etcd/issues/14533)).
- Міграція застарілих бібліотек журналювання та тегів `go-grpc-middleware` v1 на інтерцептори v2 ([#20420](https://github.com/etcd-io/etcd/pull/20420)).
- Інтерцептори gRPC OpenTelemetry було оновлено до `otelgrpc` v0.61.0, замінивши застарілі `UnaryServerInterceptor` та `StreamServerInterceptor` на `NewServerHandler` ([#20017](https://github.com/etcd-io/etcd/pull/20017)).

Якщо ви вбудовуєте etcd як бібліотеку, збираєте код проти API `clientv3` або залежите від внутрішніх пакетів, перегляньте [CHANGELOG](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.7.md) перед оновленням.

### Видалені прапорці {#flags-removed}

Усі застарілі прапорці `--experimental-*` було видалено у v3.7 ([#19959](https://github.com/etcd-io/etcd/pull/19959)). У v3.6 кожен з них було замінено або неекспериментальним прапорцем з тією ж назвою, або записом `--feature-gates`. Якщо будь-який з них все ще встановлений, замініть їх на еквівалент v3.6 **перед** переходом на v3.7, інакше процес v3.7 не зможе запуститися.

```diff
-etcd --experimental-bootstrap-defrag-threshold-megabytes
-etcd --experimental-compact-hash-check-enabled
-etcd --experimental-compact-hash-check-time
-etcd --experimental-compaction-batch-limit
-etcd --experimental-compaction-sleep-interval
-etcd --experimental-corrupt-check-time
-etcd --experimental-distributed-tracing-address
-etcd --experimental-distributed-tracing-instance-id
-etcd --experimental-distributed-tracing-sampling-rate
-etcd --experimental-distributed-tracing-service-name
-etcd --experimental-downgrade-check-time
-etcd --experimental-enable-distributed-tracing
-etcd --experimental-enable-lease-checkpoint
-etcd --experimental-enable-lease-checkpoint-persist
-etcd --experimental-initial-corrupt-check
-etcd --experimental-memory-mlock
-etcd --experimental-peer-skip-client-san-verification
-etcd --experimental-snapshot-catchup-entries
-etcd --experimental-stop-grpc-service-on-defrag
-etcd --experimental-txn-mode-write-with-shared-buffer
-etcd --experimental-warning-apply-duration
-etcd --experimental-warning-unary-request-duration
-etcd --experimental-watch-progress-notify-interval
```

Зверніться до [посібника з оновлення з v3.5 до v3.6](/docs/v3.8/upgrades/upgrade_3_6/) для відповідності кожного видаленого прапорця його неекспериментальному еквіваленту або запису `--feature-gates`.

### Додані прапорці {#flags-added}

Немає.

### Прапорці з новими стандартними значеннями {#flags-with-new-defaults}

Немає.

### Контрольні списки оновлення сервера {#server-upgrade-checklists}

#### Вимоги до оновлення {#upgrade-requirements}

Щоб оновити наявне розгортання etcd до v3.7, кластер, що працює, повинен бути версії v3.6.11 або вище. Якщо він на старішій мінорній версії, спочатку [оновіть до v3.6](/docs/v3.8/upgrades/upgrade_3_6/); etcd підтримує оновлення лише на одну мінорну версію за раз.

Крім того, щоб забезпечити плавне поступове оновлення, кластер, що працює, повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед оновленням etcd завжди тестуйте сервіси, що залежать від etcd, у тестовому середовищі перед розгортанням оновлення у виробничому середовищі.

Перед початком [завантажте резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup). Якщо щось піде не так під час оновлення, можна використовувати цю резервну копію для [відкату](#rollback) до наявної версії etcd.

#### Змішані версії {#mixed-versions}

Під час оновлення кластер etcd підтримує змішані версії учасників etcd і працює з протоколом найнижчої загальної версії. Кластер вважається оновленим лише після того, як усі його учасники будуть оновлені до версії v3.7. Внутрішньо учасники etcd ведуть переговори один з одним, щоб визначити загальну версію кластера, яка контролює звітну версію та підтримувані функції.

#### Відкат {#rollback}

Перед оновленням вашого кластера etcd створіть та [завантажте резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup) вашого кластера etcd. Цей знімок можна використовувати для відновлення кластера до стану до оновлення за потреби. Якщо під час оновлення виникають проблеми, спочатку визначте та усуньте їхню першопричину. Якщо кластер все ще перебуває у стані змішаних версій, де хоча б один учасник залишається на v3.6, можна або замінити двійковий файл або образ на попередню версію v3.6, або безпосередньо відновити кластер за допомогою знімка. У цьому змішаному стані кластер продовжує працювати як кластер v3.6, дозволяючи відкат без дотримання формальної процедури зниження версії.

Однак, щойно всі учасники будуть оновлені до v3.7, кластер вважається повністю оновленим, і відкат за допомогою двійкових файлів більше неможливий. У цьому випадку єдиними варіантами відновлення є відновлення зі знімка, створеного до оновлення, або дотримання офіційного [посібника зі зниження версії](/docs/v3.8/downgrades/downgrading-etcd/), якщо ваше оновлення піде не так.

### Процедура оновлення {#upgrade-procedure}

Цей приклад показує, як оновити кластер etcd v3.6 з 3 учасниками, що працює на локальній машині. Вивід нижче отримано з реального запуску etcd v3.6.12 та etcd v3.7.0-rc.0 на одному хості з трьома портами loopback.

#### Крок 1: перевірте вимоги до оновлення {#step-1-check-upgrade-requirements}

Чи кластер справний і працює на v3.6.11 або пізнішій версії?

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint health
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 7.681459ms
localhost:22379 is healthy: successfully committed proposal: took = 7.691750ms
localhost:32379 is healthy: successfully committed proposal: took = 7.698000ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.6.12","etcdcluster":"3.6.0","storage":"3.6.0"}
COMMENT
```

#### Крок 2: завантажте резервну копію знімка з лідера {#step-2-download-snapshot-backup-from-leader}

[Завантажте резервну копію знімка](/docs/v3.8/op-guide/maintenance/#snapshot-backup), щоб забезпечити шлях для зниження версії у разі виникнення проблем.

Лідер etcd гарантовано має останні дані застосунків, тому отримайте знімок з лідера:

```bash
for p in 2379 22379 32379; do
  echo -n "localhost:$p leader="
  curl -sL http://localhost:$p/metrics | grep "^etcd_server_is_leader " | awk '{print $2}'
done
<<COMMENT
localhost:2379 leader=1
localhost:22379 leader=0
localhost:32379 leader=0
COMMENT

etcdctl --endpoints=localhost:2379 snapshot save backup.db
<<COMMENT
{"level":"info","ts":"2026-06-02T07:01:41.863225+0300","caller":"snapshot/v3_snapshot.go:83","msg":"created temporary db file","path":"backup.db.part"}
{"level":"info","ts":"2026-06-02T07:01:41.866451+0300","logger":"client","caller":"v3@v3.6.12/maintenance.go:236","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":"2026-06-02T07:01:41.874080+0300","caller":"snapshot/v3_snapshot.go:96","msg":"fetching snapshot","endpoint":"localhost:2379"}
{"level":"info","ts":"2026-06-02T07:01:41.877203+0300","caller":"snapshot/v3_snapshot.go:111","msg":"fetched snapshot","endpoint":"localhost:2379","size":"98 kB","took":"13.822583ms","etcd-version":"3.6.0"}
{"level":"info","ts":"2026-06-02T07:01:41.877303+0300","caller":"snapshot/v3_snapshot.go:121","msg":"saved","path":"backup.db"}
Snapshot saved at backup.db
Server version 3.6.0
COMMENT
```

#### Крок 3: зупиніть один наявний сервер etcd {#step-3-stop-one-existing-etcd-server}

Коли кожен процес etcd зупиняється, очікувані помилки будуть записані іншими учасниками кластера. Це нормально, оскільки зʼєднання учасника кластера було (тимчасово) розірвано. Лідер передасть лідерство перед завершенням роботи:

```bash
{"level":"info","ts":"2026-06-02T07:01:54.949299+0300","caller":"etcdserver/server.go:1274","msg":"leadership transfer finished","local-member-id":"7339c4e5e833c029","old-leader-member-id":"7339c4e5e833c029","new-leader-member-id":"b548c2511513015","took":"101.052625ms"}
{"level":"info","ts":"2026-06-02T07:01:54.949369+0300","caller":"etcdserver/server.go:2349","msg":"server has stopped; stopping cluster version's monitor"}
{"level":"info","ts":"2026-06-02T07:01:55.503219+0300","caller":"embed/etcd.go:626","msg":"stopped serving peer traffic","address":"127.0.0.1:2380"}
```

#### Крок 4: перезапустіть сервер etcd з тією ж конфігурацією {#step-4-restart-the-etcd-server-with-same-configuration}

Перезапустіть сервер etcd з тією ж конфігурацією, але з новим двійковим файлом etcd.

```diff
-etcd-old --name ${name} \
+etcd-new --name ${name} \
  --data-dir /path/to/${name}.etcd \
  --listen-client-urls http://localhost:2379 \
  --advertise-client-urls http://localhost:2379 \
  --listen-peer-urls http://localhost:2380 \
  --initial-advertise-peer-urls http://localhost:2380 \
  --initial-cluster s1=http://localhost:2380,s2=http://localhost:22380,s3=http://localhost:32380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new
```

Новий etcd v3.7 опублікує свою інформацію в кластері. На цьому етапі кластер все ще працює за протоколом v3.6, який є найнижчою загальною версією.

> `{"level":"info","ts":"2026-06-02T07:01:58.920780+0300","caller":"membership/cluster.go:296","msg":"set cluster version from store","cluster-version":"3.6"}`
>
> `{"level":"info","ts":"2026-06-02T07:01:58.979186+0300","caller":"etcdserver/server.go:1828","msg":"published local member to cluster through raft","local-member-id":"7339c4e5e833c029","local-member-attributes":"{Name:s1 ClientURLs:[http://localhost:2379]}","cluster-id":"7dee9ba76d59ed53","publish-timeout":"7s"}`

Переконайтеся, що кожен учасник, а потім і весь кластер, стає справним з новим двійковим файлом etcd v3.7:

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint status -w table
<<COMMENT
+-----------------+------------------+------------+-----------------+---------+--------+-----------+
|    ENDPOINT     |        ID        |  VERSION   | STORAGE VERSION | DB SIZE | LEADER | RAFT TERM |
+-----------------+------------------+------------+-----------------+---------+--------+-----------+
|  localhost:2379 | 7339c4e5e833c029 | 3.7.0-rc.0 |           3.6.0 |   98 kB |  false |         3 |
| localhost:22379 | 729934363faa4a24 |     3.6.12 |           3.6.0 |   98 kB |  false |         3 |
| localhost:32379 |  b548c2511513015 |     3.6.12 |           3.6.0 |   98 kB |   true |         3 |
+-----------------+------------------+------------+-----------------+---------+--------+-----------+
COMMENT
```

Неоновлені учасники та оновлений учасник будуть записувати повідомлення про стан змішаних версій, доки весь кластер не буде оновлено. Це очікувано і припиниться після того, як усі учасники кластера etcd будуть оновлені до v3.7.

#### Крок 5: повторіть *крок 3* та *крок 4* для решти учасників {#step-5-repeat-step-3-and-step-4-for-rest-of-the-members}

Коли всі учасники оновлені, кластер повідомить про успішне оновлення до v3.7:

> `{"level":"info","ts":"2026-06-02T07:02:36.054783+0300","caller":"etcdserver/server.go:2311","msg":"updating cluster version using v3 API","from":"3.6","to":"3.7"}`
>
> `{"level":"info","ts":"2026-06-02T07:02:36.059345+0300","caller":"membership/cluster.go:593","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"7339c4e5e833c029","from":"3.6","to":"3.7"}`
>
> `{"level":"info","ts":"2026-06-02T07:02:36.059409+0300","caller":"etcdserver/server.go:2326","msg":"cluster version is updated","cluster-version":"3.7"}`

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint health
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 550.833µs
localhost:32379 is healthy: successfully committed proposal: took = 733.458µs
localhost:22379 is healthy: successfully committed proposal: took = 714.416µs
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
```

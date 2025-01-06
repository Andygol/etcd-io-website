---
title: Оновлення etcd з 3.4 до 3.5
weight: 6650
description: Процеси, контрольні списки та примітки щодо оновлення etcd з 3.4 до 3.5
---

У загальному випадку, оновлення з etcd 3.4 до 3.5 може бути безперервним, поступовим оновленням:

- по черзі зупиняйте процеси etcd v3.4 і замінюйте їх процесами etcd v3.5
- після запуску всіх процесів v3.5 нові функції в v3.5 стають доступними для кластера

Перед тим як [почати оновлення](#upgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки оновлення {#upgrade-checklists}

**ПРИМІТКА:** При [міграції з v2 без даних v3](https://github.com/etcd-io/etcd/issues/9480), сервер etcd v3.2+ панікує, коли etcd відновлюється з наявних знімків, але немає файлу v3 `ETCD_DATA_DIR/member/snap/db`. Це відбувається, коли сервер мігрував з v2 без попередніх даних v3. Це також запобігає випадковій втраті даних v3 (наприклад, файл `db` міг бути переміщений). etcd вимагає, щоб після міграції v3 можна було продовжувати лише з даними v3. Не оновлюйте до новіших версій v3, поки сервер v3.0 не містить даних v3.

**ПРИМІТКА:** Якщо ваш кластер увімкнув автентифікацію, поступове оновлення з версії 3.4 або старішої не підтримується, оскільки в 3.5 [змінюється формат записів WAL, повʼязаних з автентифікацією](https://github.com/etcd-io/etcd/pull/11943).

Основні зміни в 3.5.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_db_total_size_in_bytes` {#deprecated-etcd_debugging_mvcc_db_total_size_in_bytes-prometheus-metrics}

v3.5 підвищила метрики Prometheus `etcd_debugging_mvcc_db_total_size_in_bytes` до `etcd_mvcc_db_total_size_in_bytes`, щоб заохотити моніторинг сховища etcd. І у v3.5 визнано повністю застарілою `etcd_debugging_mvcc_db_total_size_in_bytes`.

```diff
-etcd_debugging_mvcc_db_total_size_in_bytes
+etcd_mvcc_db_total_size_in_bytes
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_put_total` {#deprecated-etcd_debugging_mvcc_put_total-prometheus-metrics}

v3.5 підвищила метрики Prometheus `etcd_debugging_mvcc_put_total` до `etcd_mvcc_put_total`, щоб заохотити моніторинг сховища etcd. І в v3.5 повністю застаріла `etcd_debugging_mvcc_put_total`.

```diff
-etcd_debugging_mvcc_put_total
+etcd_mvcc_put_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_delete_total` {#deprecated-etcd_debugging_mvcc_delete_total-prometheus-metrics}

v3.5 підвищила метрики Prometheus `etcd_debugging_mvcc_delete_total` до `etcd_mvcc_delete_total`, щоб заохотити моніторинг сховища etcd. І в v3.5 повністю застаріла `etcd_debugging_mvcc_delete_total`.

```diff
-etcd_debugging_mvcc_delete_total
+etcd_mvcc_delete_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_txn_total` {#deprecated-etcd_debugging_mvcc_txn_total-prometheus-metrics}

v3.5 підвищила метрики Prometheus `etcd_debugging_mvcc_txn_total` до `etcd_mvcc_txn_total`, щоб заохотити моніторинг сховища etcd. І в v3.5 повністю застаріла `etcd_debugging_mvcc_txn_total`.

```diff
-etcd_debugging_mvcc_txn_total
+etcd_mvcc_txn_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_range_total` {#deprecated-etcd_debugging_mvcc_range_total-prometheus-metrics}

v3.5 підвищила метрики Prometheus `etcd_debugging_mvcc_range_total` до `etcd_mvcc_range_total`, щоб заохотити моніторинг сховища etcd. І в v3.5 повністю застаріла `etcd_debugging_mvcc_range_total`.

```diff
-etcd_debugging_mvcc_range_total
+etcd_mvcc_range_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілий `etcd --logger capnslog` {#deprecated-etcd-logger-capnslog}

v3.4 стандартно використовує `--logger=zap`, щоб підтримувати кілька виводів журналу та структуроване ведення журналу.

**`etcd --logger=capnslog` застаріло у v3.5**, і тепер стандартно `--logger=zap`.

```diff
-etcd --logger=capnslog
+etcd --logger=zap --log-outputs=stderr

+# щоб записувати журнали одночасно в stderr і файл a.log
+etcd --logger=zap --log-outputs=stderr,a.log
```

v3.4 додає підтримку `etcd --logger=zap` для структурованого ведення журналу та кількох виводів журналу. Основна мотивація полягає в тому, щоб сприяти автоматизованому моніторингу etcd, а не переглядати журнали сервера, коли він починає ламатися. Майбутній розвиток зробить журнали etcd якомога меншими та полегшить моніторинг etcd за допомогою метрик та сповіщень. **`etcd --logger=capnslog` буде застарілим у v3.5.**

#### Застарілий `etcd --log-output` {#deprecated-etcd-log-output}

v3.4 перейменувала [`etcd --log-output` на `--log-outputs`](https://github.com/etcd-io/etcd/pull/9624), щоб підтримувати кілька виводів журналу.

**`etcd --log-output` застаріло у v3.5.**

```diff
-etcd --log-output=stderr
+etcd --log-outputs=stderr
```

#### Застарілий прапорець `etcd --debug` (тепер `--log-level=debug`) {#deprecated-etcd-debug-flag-now-log-level-debug}

**Прапорець `etcd --debug` застарілий.**

```diff
-etcd --debug
+etcd --log-level debug
```

#### Застарілий прапорець `etcd --log-package-levels` {#deprecated-etcd-log-package-levels}

**Прапорець `etcd --log-package-levels` для `capnslog` застарілий.**

Тепер стандартно використовується **`etcd --logger=zap`**.

```diff
-etcd --log-package-levels 'etcdmain=CRITICAL,etcdserver=DEBUG'
+etcd --logger=zap --log-outputs=stderr
```

#### Застарілий `[CLIENT-URL]/config/local/log` {#deprecated-client-url-config-local-log}

**Точка доступу `/config/local/log` застаріває у v3.5, як і прапорець `etcd --log-package-levels`.**

```diff
-$ curl http://127.0.0.1:2379/config/local/log -XPUT -d '{"Level":"DEBUG"}'
-# увімкнено ведення журналу налагодження
```

#### Змінені HTTP кінцеві точки gRPC шлюзу (застарілий `/v3beta`) {#changed-grpc-gateway-http-endpoints-deprecated-v3beta}

До

```bash
curl -L http://localhost:2379/v3beta/kv/put \
  -X POST -d '{"key": "Zm9v", "value": "YmFy"}'
```

Після

```bash
curl -L http://localhost:2379/v3/kv/put \
  -X POST -d '{"key": "Zm9v", "value": "YmFy"}'
```

`/v3beta` було видалено у випуску 3.5.

### Контрольні списки оновлення сервера {#server-upgrade-checklists}

#### Вимоги до оновлення {#upgrade-requirements}

Щоб оновити наявне розгортання etcd до 3.5, кластер, що працює, повинен бути версії 3.4 або вище. Якщо це версія до 3.4, будь ласка, [оновіть до 3.4](/docs/v3.5/upgrades/upgrade_3_3/) перед оновленням до 3.5.

Крім того, щоб забезпечити плавне поступове оновлення, кластер, що працює, повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед оновленням etcd завжди тестуйте служби, що залежать від etcd, у тестовому середовищі перед розгортанням оновлення у промисловому середовищі.

Перед початком [завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup). Якщо щось піде не так під час оновлення, можна використовувати цю резервну копію для [пониження версії](#downgrade) до поточної версії etcd. Зверніть увагу, що команда `snapshot` створює резервну копію лише даних v3. Для даних v2 див. [резервне копіювання сховища v2](/docs/v2.3/admin_guide#backing-up-the-datastore).

#### Змішані версії {#mixed-versions}

Під час оновлення кластер etcd підтримує змішані версії учасників etcd і працює з протоколом найнижчої загальної версії. Кластер вважається оновленим лише після того, як усі його учасники будуть оновлені до версії 3.5. Внутрішньо учасники etcd ведуть переговори один з одним, щоб визначити загальну версію кластера, яка контролює звітну версію та підтримувані функції.

#### Обмеження {#limitations}

Примітка: Якщо кластер містить лише дані v3 і не містить даних v2, це обмеження не застосовується.

Якщо кластер обслуговує набір даних v2 розміром понад 50 МБ, кожному новому оновленому учаснику може знадобитися до двох хвилин, щоб наздогнати поточний кластер. Перевірте розмір останнього знімка, щоб оцінити загальний розмір даних. Іншими словами, найкраще зачекати 2 хвилини між оновленням кожного учасника.

Для набагато більшого загального розміру даних, 100 МБ або більше, цей одноразовий процес може зайняти ще більше часу. Адміністратори дуже великих кластерів etcd такого масштабу можуть звернутися до [команди etcd][etcd-contact] перед оновленням, і ми будемо раді надати поради щодо процедури.

#### Пониження версії {#downgrade}

Якщо всі учасники були оновлені до v3.5, кластер буде оновлено до v3.5, і пониження версії з цього завершеного стану **неможливе**. Однак, якщо будь-який окремий учасник все ще є v3.4, кластер і його операції залишаються "v3.4", і з цього змішаного стану кластера можна повернутися до використання двійкового файлу v3.4 etcd на всіх учасниках.

Будь ласка, [завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup), щоб зробити можливим пониження версії кластера навіть після його повного оновлення.

### Процедура оновлення {#upgrade-procedure}

Цей приклад показує, як оновити кластер etcd v3.4 з 3 учасниками, що працює на локальній машині.

#### Крок 1: перевірте вимоги до оновлення {#step-1-check-upgrade-requirements}

Чи кластер справний і працює на v3.4.x?

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint health
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 2.118638ms
localhost:22379 is healthy: successfully committed proposal: took = 3.631388ms
localhost:32379 is healthy: successfully committed proposal: took = 2.157051ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.4.0","etcdcluster":"3.4.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.4.0","etcdcluster":"3.4.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.4.0","etcdcluster":"3.4.0"}
COMMENT
```

#### Крок 2: завантажте резервну копію знімка з лідера {#step-2-download-snapshot-backup-from-leader}

[Завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup), щоб забезпечити шлях до пониження версії у разі виникнення проблем.

лідер etcd гарантовано має останні дані застосунків, тому отримайте знімок з лідера:

```bash
curl -sL http://localhost:2379/metrics | grep etcd_server_is_leader
<<COMMENT
# HELP etcd_server_is_leader Whether or not this member is a leader. 1 if is, 0 otherwise.
# TYPE etcd_server_is_leader gauge
etcd_server_is_leader 1
COMMENT

curl -sL http://localhost:22379/metrics | grep etcd_server_is_leader
<<COMMENT
etcd_server_is_leader 0
COMMENT

curl -sL http://localhost:32379/metrics | grep etcd_server_is_leader
<<COMMENT
etcd_server_is_leader 0
COMMENT

etcdctl --endpoints=localhost:2379 snapshot save backup.db
<<COMMENT
{"level":"info","ts":1526585787.148433,"caller":"snapshot/v3_snapshot.go:109","msg":"created temporary db file","path":"backup.db.part"}
{"level":"info","ts":1526585787.1485257,"caller":"snapshot/v3_snapshot.go:120","msg":"fetching snapshot","endpoint":"localhost:2379"}
{"level":"info","ts":1526585787.1519694,"caller":"snapshot/v3_snapshot.go:133","msg":"fetched snapshot","endpoint":"localhost:2379","took":0.003502721}
{"level":"info","ts":1526585787.1520295,"caller":"snapshot/v3_snapshot.go:142","msg":"saved","path":"backup.db"}
Snapshot saved at backup.db
COMMENT
```

#### Крок 3: зупиніть один наявний сервер etcd {#step-3-stop-one-existing-etcd-server}

Коли кожен процес etcd зупиняється, очікувані помилки будуть записані іншими учасниками кластера. Це нормально, оскільки зʼєднання учасника кластера було (тимчасово) розірвано:

```bash
{"level":"info","ts":1526587281.2001143,"caller":"etcdserver/server.go:2249","msg":"updating cluster version","from":"3.0","to":"3.4"}
{"level":"info","ts":1526587281.2010646,"caller":"membership/cluster.go:473","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"7339c4e5e833c029","from":"3.0","from":"3.4"}
{"level":"info","ts":1526587281.2012327,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.4"}
{"level":"info","ts":1526587281.2013083,"caller":"etcdserver/server.go:2272","msg":"cluster version is updated","cluster-version":"3.4"}



^C{"level":"info","ts":1526587299.0717514,"caller":"osutil/interrupt_unix.go:63","msg":"received signal; shutting down","signal":"interrupt"}
{"level":"info","ts":1526587299.0718873,"caller":"embed/etcd.go:285","msg":"closing etcd server","name":"s1","data-dir":"/tmp/etcd/s1","advertise-peer-urls":["http://localhost:2380"],"advertise-client-urls":["http://localhost:2379"]}
{"level":"info","ts":1526587299.0722554,"caller":"etcdserver/server.go:1341","msg":"leadership transfer starting","local-member-id":"7339c4e5e833c029","current-leader-member-id":"7339c4e5e833c029","transferee-member-id":"729934363faa4a24"}
{"level":"info","ts":1526587299.0723994,"caller":"raft/raft.go:1107","msg":"7339c4e5e833c029 [term 3] starts to transfer leadership to 729934363faa4a24"}
{"level":"info","ts":1526587299.0724802,"caller":"raft/raft.go:1113","msg":"7339c4e5e833c029 sends MsgTimeoutNow to 729934363faa4a24 immediately as 729934363faa4a24 already has up-to-date log"}
{"level":"info","ts":1526587299.0737045,"caller":"raft/raft.go:797","msg":"7339c4e5e833c029 [term: 3] received a MsgVote message with higher term from 729934363faa4a24 [term: 4]"}
{"level":"info","ts":1526587299.0737681,"caller":"raft/raft.go:656","msg":"7339c4e5e833c029 became follower at term 4"}
{"level":"info","ts":1526587299.073831,"caller":"raft/raft.go:882","msg":"7339c4e5e833c029 [logterm: 3, index: 9, vote: 0] cast MsgVote for 729934363faa4a24 [logterm: 3, index: 9] at term 4"}
{"level":"info","ts":1526587299.0738947,"caller":"raft/node.go:312","msg":"raft.node: 7339c4e5e833c029 lost leader 7339c4e5e833c029 at term 4"}
{"level":"info","ts":1526587299.0748374,"caller":"raft/node.go:306","msg":"raft.node: 7339c4e5e833c029 elected leader 729934363faa4a24 at term 4"}
{"level":"info","ts":1526587299.1726425,"caller":"etcdserver/server.go:1362","msg":"leadership transfer finished","local-member-id":"7339c4e5e833c029","old-leader-member-id":"7339c4e5e833c029","new-leader-member-id":"729934363faa4a24","took":0.100389359}
{"level":"info","ts":1526587299.1728148,"caller":"rafthttp/peer.go:333","msg":"stopping remote peer","remote-peer-id":"b548c2511513015"}
{"level":"warn","ts":1526587299.1751974,"caller":"rafthttp/stream.go:291","msg":"closed TCP streaming connection with remote peer","stream-writer-type":"stream MsgApp v2","remote-peer-id":"b548c2511513015"}
{"level":"warn","ts":1526587299.1752589,"caller":"rafthttp/stream.go:301","msg":"stopped TCP streaming connection with remote peer","stream-writer-type":"stream MsgApp v2","remote-peer-id":"b548c2511513015"}
{"level":"warn","ts":1526587299.177348,"caller":"rafthttp/stream.go:291","msg":"closed TCP streaming connection with remote peer","stream-writer-type":"stream Message","remote-peer-id":"b548c2511513015"}
{"level":"warn","ts":1526587299.1774004,"caller":"rafthttp/stream.go:301","msg":"stopped TCP streaming connection with remote peer","stream-writer-type":"stream Message","remote-peer-id":"b548c2511513015"}
{"level":"info","ts":1526587299.177515,"caller":"rafthttp/pipeline.go:86","msg":"stopped HTTP pipelining with remote peer","local-member-id":"7339c4e5e833c029","remote-peer-id":"b548c2511513015"}
{"level":"warn","ts":1526587299.1777067,"caller":"rafthttp/stream.go:436","msg":"lost TCP streaming connection with remote peer","stream-reader-type":"stream MsgApp v2","local-member-id":"7339c4e5e833c029","remote-peer-id":"b548c2511513015","error":"read tcp 127.0.0.1:34636->127.0.0.1:32380: use of closed network connection"}
{"level":"info","ts":1526587299.1778402,"caller":"rafthttp/stream.go:459","msg":"stopped stream reader with remote peer","stream-reader-type":"stream MsgApp v2","local-member-id":"7339c4e5e833c029","remote-peer-id":"b548c2511513015"}
{"level":"warn","ts":1526587299.1780295,"caller":"rafthttp/stream.go:436","msg":"lost TCP streaming connection with remote peer","stream-reader-type":"stream Message","local-member-id":"7339c4e5e833c029","remote-peer-id":"b548c2511513015","error":"read tcp 127.0.0.1:34634->127.0.0.1:32380: use of closed network connection"}
{"level":"info","ts":1526587299.1780987,"caller":"rafthttp/stream.go:459","msg":"stopped stream reader with remote peer","stream-reader-type":"stream Message","local-member-id":"7339c4e5e833c029","remote-peer-id":"b548c2511513015"}
{"level":"info","ts":1526587299.1781602,"caller":"rafthttp/peer.go:340","msg":"stopped remote peer","remote-peer-id":"b548c2511513015"}
{"level":"info","ts":1526587299.1781986,"caller":"rafthttp/peer.go:333","msg":"stopping remote peer","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":1526587299.1802843,"caller":"rafthttp/stream.go:291","msg":"closed TCP streaming connection with remote peer","stream-writer-type":"stream MsgApp v2","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":1526587299.1803446,"caller":"rafthttp/stream.go:301","msg":"stopped TCP streaming connection with remote peer","stream-writer-type":"stream MsgApp v2","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":1526587299.1824749,"caller":"rafthttp/stream.go:291","msg":"closed TCP streaming connection with remote peer","stream-writer-type":"stream Message","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":1526587299.18255,"caller":"rafthttp/stream.go:301","msg":"stopped TCP streaming connection with remote peer","stream-writer-type":"stream Message","remote-peer-id":"729934363faa4a24"}
{"level":"info","ts":1526587299.18261,"caller":"rafthttp/pipeline.go:86","msg":"stopped HTTP pipelining with remote peer","local-member-id":"7339c4e5e833c029","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":1526587299.1827736,"caller":"rafthttp/stream.go:436","msg":"lost TCP streaming connection with remote peer","stream-reader-type":"stream MsgApp v2","local-member-id":"7339c4e5e833c029","remote-peer-id":"729934363faa4a24","error":"read tcp 127.0.0.1:51482->127.0.0.1:22380: use of closed network connection"}
{"level":"info","ts":1526587299.182845,"caller":"rafthttp/stream.go:459","msg":"stopped stream reader with remote peer","stream-reader-type":"stream MsgApp v2","local-member-id":"7339c4e5e833c029","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":1526587299.1830168,"caller":"rafthttp/stream.go:436","msg":"lost TCP streaming connection with remote peer","stream-reader-type":"stream Message","local-member-id":"7339c4e5e833c029","remote-peer-id":"729934363faa4a24","error":"context canceled"}
{"level":"warn","ts":1526587299.1831107,"caller":"rafthttp/peer_status.go:65","msg":"peer became inactive","peer-id":"729934363faa4a24","error":"failed to read 729934363faa4a24 on stream Message (context canceled)"}
{"level":"info","ts":1526587299.1831737,"caller":"rafthttp/stream.go:459","msg":"stopped stream reader with remote peer","stream-reader-type":"stream Message","local-member-id":"7339c4e5e833c029","remote-peer-id":"729934363faa4a24"}
{"level":"info","ts":1526587299.1832306,"caller":"rafthttp/peer.go:340","msg":"stopped remote peer","remote-peer-id":"729934363faa4a24"}
{"level":"warn","ts":1526587299.1837125,"caller":"rafthttp/http.go:424","msg":"failed to find remote peer in cluster","local-member-id":"7339c4e5e833c029","remote-peer-id-stream-handler":"7339c4e5e833c029","remote-peer-id-from":"b548c2511513015","cluster-id":"7dee9ba76d59ed53"}
{"level":"warn","ts":1526587299.1840093,"caller":"rafthttp/http.go:424","msg":"failed to find remote peer in cluster","local-member-id":"7339c4e5e833c029","remote-peer-id-stream-handler":"7339c4e5e833c029","remote-peer-id-from":"b548c2511513015","cluster-id":"7dee9ba76d59ed53"}
{"level":"warn","ts":1526587299.1842315,"caller":"rafthttp/http.go:424","msg":"failed to find remote peer in cluster","local-member-id":"7339c4e5e833c029","remote-peer-id-stream-handler":"7339c4e5e833c029","remote-peer-id-from":"729934363faa4a24","cluster-id":"7dee9ba76d59ed53"}
{"level":"warn","ts":1526587299.1844475,"caller":"rafthttp/http.go:424","msg":"failed to find remote peer in cluster","local-member-id":"7339c4e5e833c029","remote-peer-id-stream-handler":"7339c4e5e833c029","remote-peer-id-from":"729934363faa4a24","cluster-id":"7dee9ba76d59ed53"}
{"level":"info","ts":1526587299.2056687,"caller":"embed/etcd.go:473","msg":"stopping serving peer traffic","address":"127.0.0.1:2380"}
{"level":"info","ts":1526587299.205819,"caller":"embed/etcd.go:480","msg":"stopped serving peer traffic","address":"127.0.0.1:2380"}
{"level":"info","ts":1526587299.2058413,"caller":"embed/etcd.go:289","msg":"closed etcd server","name":"s1","data-dir":"/tmp/etcd/s1","advertise-peer-urls":["http://localhost:2380"],"advertise-client-urls":["http://localhost:2379"]}
```

#### Крок 4: перезапустіть сервер etcd з тією ж конфігурацією {#step-4-restart-the-etcd-server-with-same-configuration}

Перезапустіть сервер etcd з тією ж конфігурацією, але з новим двійковим файлом etcd.

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
  --initial-cluster-state new
```

Новий etcd v3.5 опублікує свою інформацію в кластері. На цьому етапі кластер все ще працює за протоколом v3.4, який є найнижчою загальною версією.

> `{"level":"info","ts":1526586617.1647713,"caller":"membership/cluster.go:485","msg":"set initial cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"7339c4e5e833c029","cluster-version":"3.0"}`

> `{"level":"info","ts":1526586617.1648536,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.0"}`

> `{"level":"info","ts":1526586617.1649303,"caller":"membership/cluster.go:473","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"7339c4e5e833c029","from":"3.0","from":"3.4"}`

> `{"level":"info","ts":1526586617.1649797,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.4"}`

> `{"level":"info","ts":1526586617.2107732,"caller":"etcdserver/server.go:1770","msg":"published local member to cluster through raft","local-member-id":"7339c4e5e833c029","local-member-attributes":"{Name:s1 ClientURLs:[http://localhost:2379]}","request-path":"/0/members/7339c4e5e833c029/attributes","cluster-id":"7dee9ba76d59ed53","publish-timeout":7}`

Переконайтеся, що кожен учасник, а потім і весь кластер, стає здоровим із новим двійковим файлом etcd v3.5:

```bash
etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:32379 is healthy: successfully committed proposal: took = 2.337471ms
localhost:22379 is healthy: successfully committed proposal: took = 1.130717ms
localhost:2379 is healthy: successfully committed proposal: took = 2.124843ms
COMMENT
```

Неоновлені учасники будуть записувати попередження, як показано нижче, доки весь кластер не буде оновлено.

Це очікувано і припиниться після того, як усі учасники кластера etcd будуть оновлені до v3.5:

```
:41.942121 W | etcdserver: member 7339c4e5e833c029 has a higher version 3.5.0
:45.945154 W | etcdserver: the local etcd version 3.4.0 is not up-to-date
```

#### Крок 5: повторіть *крок 3* і *крок 4* для решти учасників {#step-5-repeat-step-3-and-step-4-for-rest-of-the-members}

Коли всі учасники будуть оновлені, кластер повідомить про успішне оновлення до 3.5:

Учасник 1:

> `{"level":"info","ts":1526586949.0920913,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.5"}`
> `{"level":"info","ts":1526586949.0921566,"caller":"etcdserver/server.go:2272","msg":"cluster version is updated","cluster-version":"3.5"}`

Учасник 2:

> `{"level":"info","ts":1526586949.092117,"caller":"membership/cluster.go:473","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"729934363faa4a24","from":"3.4","from":"3.5"}`
> `{"level":"info","ts":1526586949.0923078,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.5"}`

Учасник 3:

> `{"level":"info","ts":1526586949.0921423,"caller":"membership/cluster.go:473","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"b548c2511513015","from":"3.4","from":"3.5"}`
> `{"level":"info","ts":1526586949.0922918,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.5"}`

```bash
endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 492.834µs
localhost:22379 is healthy: successfully committed proposal: took = 1.015025ms
localhost:32379 is healthy: successfully committed proposal: took = 1.853077ms
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

[etcd-contact]: https://groups.google.com/g/etcd-dev

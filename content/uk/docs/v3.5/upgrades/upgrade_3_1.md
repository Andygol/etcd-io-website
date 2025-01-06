---
title: Оновлення etcd з 3.0 до 3.1
weight: 6850
description: Процеси, контрольні списки та примітки щодо оновлення etcd з 3.0 до 3.1
---

У загальному випадку, оновлення з etcd 3.0 до 3.1 може бути безперервним, поступовим оновленням:

- по черзі зупиняйте процеси etcd v3.0 і замінюйте їх процесами etcd v3.1
- після запуску всіх процесів v3.1 нові функції v3.1 стають доступними для кластера

Перед тим як [почати оновлення](#upgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки оновлення {#upgrade-checklists}

**ПРИМІТКА:** При [міграції з v2 без даних v3](https://github.com/etcd-io/etcd/issues/9480), сервер etcd v3.2+ панікує, коли відновлюється з наявних знімків, але немає файлу v3 `ETCD_DATA_DIR/member/snap/db`. Це відбувається, коли сервер мігрував з v2 без попередніх даних v3. Це також запобігає випадковій втраті даних v3 (наприклад, файл `db` міг бути переміщений). etcd вимагає, щоб після міграції v3 можна було продовжувати лише з даними v3. Не оновлюйте до новіших версій v3, поки сервер v3.0 не містить даних v3.

#### Моніторинг {#monitoring}

Наступні метрики з v3.0.x були визнані застарілими на користь [go-grpc-prometheus](https://github.com/grpc-ecosystem/go-grpc-prometheus):

- `etcd_grpc_requests_total`
- `etcd_grpc_requests_failed_total`
- `etcd_grpc_active_streams`
- `etcd_grpc_unary_requests_duration_seconds`

#### Вимоги до оновлення {#upgrade-requirements}

Щоб оновити поточне розгортання etcd до 3.1, кластер, що працює, повинен бути версії 3.0 або вище. Якщо це версія до 3.0, будь ласка, [оновіть до 3.0](docs/v3.5/upgrades/upgrade_3_0) перед оновленням до 3.1.

Також, щоб забезпечити плавне поступове оновлення, кластер, що працює, повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед оновленням etcd завжди тестуйте сервіси, що залежать від etcd, у тестовому середовищі перед розгортанням оновлення в промисловому середовищі.

Перед початком [зробіть резервну копію даних etcd](../../op-guide/maintenance/#snapshot-backup). Якщо щось піде не так з оновленням, можна використовувати цю резервну копію для [пониження версії](#downgrade) до поточної версії etcd. Зверніть увагу, що команда `snapshot` робить резервну копію лише даних v3. Для даних v2 див. [резервне копіювання сховища v2](/docs/v2.3/admin_guide#backing-up-the-datastore).

#### Змішані версії {#mixed-versions}

Під час оновлення кластер etcd підтримує змішані версії членів etcd і працює з протоколом найнижчої загальної версії. Кластер вважається оновленим лише після того, як всі його члени будуть оновлені до версії 3.1. Внутрішньо члени etcd домовляються між собою, щоб визначити загальну версію кластера, яка контролює версію, про яку повідомляється, та підтримувані функції.

#### Обмеження {#limitations}

Примітка: Якщо кластер містить лише дані v3 і не містить даних v2, це обмеження не застосовується.

Якщо кластер обслуговує набір даних v2 розміром понад 50 МБ, кожен новий оновлений член може зайняти до двох хвилин, щоб наздогнати поточний кластер. Перевірте розмір останнього знімка, щоб оцінити загальний розмір даних. Іншими словами, найкраще почекати 2 хвилини між оновленням кожного члена.

Для набагато більшого загального розміру даних, 100 МБ або більше, цей одноразовий процес може зайняти ще більше часу. Адміністратори дуже великих кластерів etcd такого масштабу можуть звернутися до [команди etcd][etcd-contact] перед оновленням, і ми будемо раді надати поради щодо процедури.

#### Пониження версії {#downgrade}

Якщо всі члени були оновлені до v3.1, кластер буде оновлено до v3.1, і пониження версії з цього завершеного стану **неможливе**. Однак, якщо будь-який окремий член все ще є v3.0, кластер і його операції залишаються "v3.0", і з цього змішаного стану кластера можна повернутися до використання двійкового файлу etcd v3.0 на всіх членах.

Будь ласка, [зробіть резервну копію теки даних](../../op-guide/maintenance#snapshot-backup) всіх членів etcd, щоб зробити можливим пониження версії кластера навіть після його повного оновлення.

### Процедура оновлення {#upgrade-procedure}

Цей приклад показує, як оновити кластер etcd з 3 членами v3.0, що працює на локальній машині.

#### 1. Перевірте вимоги до оновлення {#check-upgrade-requirements}

Чи кластер справний і працює на v3.0.x?

```sh
$ ETCDCTL_API=3 etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:2379 is healthy: successfully committed proposal: took = 6.600684ms
localhost:22379 is healthy: successfully committed proposal: took = 8.540064ms
localhost:32379 is healthy: successfully committed proposal: took = 8.763432ms

$ curl http://localhost:2379/version
{"etcdserver":"3.0.16","etcdcluster":"3.0.0"}
```

#### 2. Зупиніть поточний процес etcd {#stop-existing-etcd-process}

Коли кожен процес etcd зупиняється, очікувані помилки будуть зареєстровані іншими членами кластера. Це нормально, оскільки зʼєднання члена кластера було (тимчасово) розірвано:

```sh
2017-01-17 09:34:18.352662 I | raft: raft.node: 1640829d9eea5cfb elected leader 1640829d9eea5cfb at term 5
2017-01-17 09:34:18.359630 W | etcdserver: failed to reach the peerURL(http://localhost:2380) of member fd32987dcd0511e0 (Get http://localhost:2380/version: dial tcp 127.0.0.1:2380: getsockopt: connection refused)
2017-01-17 09:34:18.359679 W | etcdserver: cannot get the version of member fd32987dcd0511e0 (Get http://localhost:2380/version: dial tcp 127.0.0.1:2380: getsockopt: connection refused)
2017-01-17 09:34:18.548116 W | rafthttp: lost the TCP streaming connection with peer fd32987dcd0511e0 (stream Message writer)
2017-01-17 09:34:19.147816 W | rafthttp: lost the TCP streaming connection with peer fd32987dcd0511e0 (stream MsgApp v2 writer)
2017-01-17 09:34:34.364907 W | etcdserver: failed to reach the peerURL(http://localhost:2380) of member fd32987dcd0511e0 (Get http://localhost:2380/version: dial tcp 127.0.0.1:2380: getsockopt: connection refused)
```

На цьому етапі доцільно [зробити резервну копію даних etcd](../../op-guide/maintenance#snapshot-backup), щоб забезпечити шлях до пониження версії у разі виникнення будь-яких проблем:

```sh
$ etcdctl snapshot save backup.db
```

#### 3. Встановіть двійковий файл etcd v3.1 і запустіть новий процес etcd {#drop-in-etcd-v3-1-binary}

Новий etcd v3.1 опублікує свою інформацію в кластер:

```log
2017-01-17 09:36:00.996590 I | etcdserver: published {Name:my-etcd-1 ClientURLs:[http://localhost:2379]} to cluster 46bc3ce73049e678
```

Переконайтеся, що кожен член, а потім і весь кластер, стають справними з новим двійковим файлом etcd v3.1:

```sh
$ ETCDCTL_API=3 /etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:22379 is healthy: successfully committed proposal: took = 5.540129ms
localhost:32379 is healthy: successfully committed proposal: took = 7.321671ms
localhost:2379 is healthy: successfully committed proposal: took = 10.629901ms
```

Оновлені члени будуть реєструвати попередження, подібні до наступних, поки весь кластер не буде оновлено. Це очікувано і припиниться після того, як всі члени кластера etcd будуть оновлені до v3.1:

```log
2017-01-17 09:36:38.406268 W | etcdserver: the local etcd version 3.0.16 is not up-to-date
2017-01-17 09:36:38.406295 W | etcdserver: member fd32987dcd0511e0 has a higher version 3.1.0
2017-01-17 09:36:42.407695 W | etcdserver: the local etcd version 3.0.16 is not up-to-date
2017-01-17 09:36:42.407730 W | etcdserver: member fd32987dcd0511e0 has a higher version 3.1.0
```

#### 4. Повторіть кроки 2 та 3 для всіх інших членів {#repeat-steps}

#### 5. Завершення {#finish}

Коли всі члени будуть оновлені, кластер повідомить про успішне оновлення до 3.1:

```log
2017-01-17 09:37:03.100015 I | etcdserver: updating the cluster version from 3.0 to 3.1
2017-01-17 09:37:03.104263 N | etcdserver/membership: updated the cluster version from 3.0 to 3.1
2017-01-17 09:37:03.104374 I | etcdserver/api: enabled capabilities for version 3.1
```

```sh
$ ETCDCTL_API=3 /etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:2379 is healthy: successfully committed proposal: took = 2.312897ms
localhost:22379 is healthy: successfully committed proposal: took = 2.553476ms
localhost:32379 is healthy: successfully committed proposal: took = 2.516902ms
```

[etcd-contact]: https://groups.google.com/g/etcd-dev

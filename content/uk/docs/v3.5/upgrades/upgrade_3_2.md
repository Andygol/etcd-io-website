---
title: Оновлення etcd з 3.1 до 3.2
weight: 6800
description: Процеси, контрольні списки та примітки щодо оновлення etcd з 3.1 до 3.2
---

У загальному випадку, оновлення з etcd 3.1 до 3.2 може бути безперервним, поступовим оновленням:

- по черзі зупиняйте процеси etcd v3.1 та замінюйте їх процесами etcd v3.2
- після запуску всіх процесів v3.2 нові функції в v3.2 будуть доступні кластеру

Перед [початком оновлення](#upgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки для оновлення {#upgrade-checklists}

**ПРИМІТКА:** При [міграції з v2 без даних v3](https://github.com/etcd-io/etcd/issues/9480), сервер etcd v3.2+ панікує, коли відновлюється з наявних знімків, але немає файлу v3 `ETCD_DATA_DIR/member/snap/db`. Це відбувається, коли сервер мігрував з v2 без попередніх даних v3. Це також запобігає випадковій втраті даних v3 (наприклад, файл `db` міг бути переміщений). etcd вимагає, щоб після міграції v3 можна було продовжувати лише з даними v3. Не оновлюйте до новіших версій v3, поки сервер v3.0 не містить даних v3.

Основні зміни в 3.2.

#### Змінене стандартне значення `snapshot-count` {#changed-default-snapshot-count-value}

Вищий `--snapshot-count` утримує більше записів Raft у памʼяті до знімка, що призводить до [постійного вищого використання памʼяті](https://github.com/kubernetes/kubernetes/issues/60589#issuecomment-371977156). Оскільки лідер утримує останні записи Raft довше, повільний послідовник має більше часу на синхронізацію перед знімком лідера. `--snapshot-count` є компромісом між вищим використанням памʼяті та кращою доступністю повільних послідовників.

З версії v3.2 стандартні значення для `--snapshot-count` [змінено з 10,000 на 100,000](https://github.com/etcd-io/etcd/pull/7160).

#### Змінена залежність gRPC (>=3.2.10) {#changed-grpc-dependency}

Версія 3.2.10 або пізніша тепер вимагає [grpc/grpc-go](https://github.com/grpc/grpc-go/releases) `v1.7.5` (<=3.2.9 вимагає `v1.2.1`).

##### Застарілий `grpclog.Logger` {#deprecated-grpclog-logger}

`grpclog.Logger` застарів на користь [`grpclog.LoggerV2`](https://github.com/grpc/grpc-go/blob/master/grpclog/loggerv2.go). `clientv3.Logger` тепер є `grpclog.LoggerV2`.

До

```go
import "github.com/coreos/etcd/clientv3"
clientv3.SetLogger(log.New(os.Stderr, "grpc: ", 0))
```

Після

```go
import "github.com/coreos/etcd/clientv3"
import "google.golang.org/grpc/grpclog"
clientv3.SetLogger(grpclog.NewLoggerV2(os.Stderr, os.Stderr, os.Stderr))

// log.New вище не може бути використаний (не реалізує інтерфейс grpclog.LoggerV2)
```

##### Застарілий `grpc.ErrClientConnTimeout` {#deprecated-grpc-errclientconntimeout}

Раніше помилка `grpc.ErrClientConnTimeout` поверталася при тайм-ауті підключення клієнта. У версії 3.2 замість цього повертається `context.DeadlineExceeded` (див. [#8504](https://github.com/etcd-io/etcd/issues/8504)).

До

```go
// очікується тайм-аут підключення на ipv4 blackhole
_, err := clientv3.New(clientv3.Config{
    Endpoints:   []string{"http://254.0.0.1:12345"},
    DialTimeout: 2 * time.Second
})
if err == grpc.ErrClientConnTimeout {
	// обробка помилок
}
```

Після

```go
_, err := clientv3.New(clientv3.Config{
    Endpoints:   []string{"http://254.0.0.1:12345"},
    DialTimeout: 2 * time.Second
})
if err == context.DeadlineExceeded {
	// обробка помилок
}
```

#### Змінені обмеження розміру запиту (>=3.2.10) {#changed-maximum-request-size-limits}

Версії 3.2.10 і 3.2.11 дозволяють налаштовувати обмеження розміру запиту на стороні сервера. >=3.2.12 дозволяє налаштовувати обмеження розміру запиту як на стороні сервера, так і на стороні клієнта. У попередніх версіях (v3.2.10, v3.2.11) розмір відповіді клієнта був обмежений лише 4 МіБ.

Обмеження розміру запиту на стороні сервера можна налаштувати за допомогою прапорця `--max-request-bytes`:

```bash
# обмежує розмір запиту до 1.5 КіБ
etcd --max-request-bytes 1536

# записи клієнта, що перевищують 1.5 КіБ, будуть відхилені
etcdctl put foo [ВЕЛИКЕ ЗНАЧЕННЯ...]
# etcdserver: запит занадто великий
```

Або налаштуйте поле `embed.Config.MaxRequestBytes`:

```go
import "github.com/coreos/etcd/embed"
import "github.com/coreos/etcd/etcdserver/api/v3rpc/rpctypes"

// обмежує запити до 5 МіБ
cfg := embed.NewConfig()
cfg.MaxRequestBytes = 5 * 1024 * 1024

// записи клієнта, що перевищують 5 МіБ, будуть відхилені
_, err := cli.Put(ctx, "foo", [ВЕЛИКЕ ЗНАЧЕННЯ...])
err == rpctypes.ErrRequestTooLarge
```

**Якщо не вказано, обмеження стандартно на стороні сервера становить 1.5 МіБ**.

Обмеження розміру запиту на стороні клієнта повинні бути налаштовані на основі обмежень на стороні сервера.

```bash
# обмежує розмір запиту до 1 МіБ
etcd --max-request-bytes 1048576
```

```go
import "github.com/coreos/etcd/clientv3"

cli, _ := clientv3.New(clientv3.Config{
    Endpoints: []string{"127.0.0.1:2379"},
    MaxCallSendMsgSize: 2 * 1024 * 1024,
    MaxCallRecvMsgSize: 3 * 1024 * 1024,
})


// записи клієнта, що перевищують "--max-request-bytes", будуть відхилені сервером etcd
_, err := cli.Put(ctx, "foo", strings.Repeat("a", 1*1024*1024+5))
err == rpctypes.ErrRequestTooLarge


// записи клієнта, що перевищують "MaxCallSendMsgSize", будуть відхилені на стороні клієнта
_, err = cli.Put(ctx, "foo", strings.Repeat("a", 5*1024*1024))
err.Error() == "rpc error: code = ResourceExhausted desc = grpc: trying to send message larger than max (5242890 vs. 2097152)"


// деякі записи під обмеженнями
for i := range []int{0,1,2,3,4} {
    _, err = cli.Put(ctx, fmt.Sprintf("foo%d", i), strings.Repeat("a", 1*1024*1024-500))
    if err != nil {
        panic(err)
    }
}
// читання клієнта, що перевищує "MaxCallRecvMsgSize", буде відхилено на стороні клієнта
_, err = cli.Get(ctx, "foo", clientv3.WithPrefix())
err.Error() == "rpc error: code = ResourceExhausted desc = grpc: received message larger than max (5240509 vs. 3145728)"
```

**Якщо не вказано, стандартне обмеження на стороні клієнта становить 2 МіБ (1.5 МіБ + байти накладних витрат gRPC) і обмеження на отримання до `math.MaxInt32`**. Будь ласка, дивіться [документацію clientv3](https://pkg.go.dev/github.com/etcd-io/etcd/clientv3#Config) для отримання додаткової інформації.

#### Змінені обгортки клієнтів gRPC {#changed-raw-grpc-client-wrappers}

Версія 3.2.12 або пізніша змінює сигнатури функцій обгортки клієнтів gRPC `clientv3`. Ця зміна була необхідна для підтримки [власних `grpc.CallOption` на обмеження розміру повідомлень](https://github.com/etcd-io/etcd/pull/9047).

До і після

```diff
-func NewKVFromKVClient(remote pb.KVClient) KV {
+func NewKVFromKVClient(remote pb.KVClient, c *Client) KV {

-func NewClusterFromClusterClient(remote pb.ClusterClient) Cluster {
+func NewClusterFromClusterClient(remote pb.ClusterClient, c *Client) Cluster {

-func NewLeaseFromLeaseClient(remote pb.LeaseClient, keepAliveTimeout time.Duration) Lease {
+func NewLeaseFromLeaseClient(remote pb.LeaseClient, c *Client, keepAliveTimeout time.Duration) Lease {

-func NewMaintenanceFromMaintenanceClient(remote pb.MaintenanceClient) Maintenance {
+func NewMaintenanceFromMaintenanceClient(remote pb.MaintenanceClient, c *Client) Maintenance {

-func NewWatchFromWatchClient(wc pb.WatchClient) Watcher {
+func NewWatchFromWatchClient(wc pb.WatchClient, c *Client) Watcher {
```

#### Змінений API `clientv3.Lease.TimeToLive` {#changed-clientv3-lease-timetolive-api}

Раніше API `clientv3.Lease.TimeToLive` повертав `lease.ErrLeaseNotFound` для відсутнього ідентифікатора оренди. У версії 3.2 замість цього повертається TTL=-1 у відповіді та без помилки (див. [#7305](https://github.com/etcd-io/etcd/pull/7305)).

До

```go
// коли leaseID не існує
resp, err := TimeToLive(ctx, leaseID)
resp == nil
err == lease.ErrLeaseNotFound
```

Після

```go
// коли leaseID не існує
resp, err := TimeToLive(ctx, leaseID)
resp.TTL == -1
err == nil
```

#### Переміщено `clientv3.NewFromConfigFile` до `clientv3.yaml.NewConfig` {#moved-clientv3-newfromconfigfile-to-clientv3-yaml-newconfig}

`clientv3.NewFromConfigFile` переміщено до `yaml.NewConfig`.

До

```go
import "github.com/coreos/etcd/clientv3"
clientv3.NewFromConfigFile
```

Після

```go
import clientv3yaml "github.com/coreos/etcd/clientv3/yaml"
clientv3yaml.NewConfig
```

#### Зміна в `--listen-peer-urls` та `--listen-client-urls` {#change-in-listen-peer-urls-and-listen-client-urls}

Версія 3.2 тепер відхиляє доменні імена для `--listen-peer-urls` та `--listen-client-urls` (версія 3.1 лише виводить попередження), оскільки доменне імʼя є недійсним для привʼязки до мережевого інтерфейсу. Переконайтеся, що ці URL-адреси правильно відформатовані як `scheme://IP:port`.

Дивіться [issue #6336](https://github.com/etcd-io/etcd/issues/6336) для отримання додаткової інформації.

### Контрольні списки для оновлення сервера {#server-upgrade-checklists}

#### Вимоги до оновлення {#upgrade-requirements}

Щоб оновити поточне розгортання etcd до версії 3.2, працюючий кластер повинен бути версії 3.1 або вище. Якщо це версія до 3.1, будь ласка, [оновіть до версії 3.1](/docs/v3.5/upgrades/upgrade_3_1) перед оновленням до версії 3.2.

Також, щоб забезпечити плавне поступове оновлення, працюючий кластер повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед оновленням etcd завжди тестуйте сервіси, що залежать від etcd, у тестовому середовищі перед розгортанням оновлення у промисловому середовищі.

Перед початком [зробіть резервну копію даних etcd](../../op-guide/maintenance#snapshot-backup). Якщо щось піде не так під час оновлення, можна використовувати цю резервну копію для [пониження версії](#downgrade) до поточної версії etcd. Зверніть увагу, що команда `snapshot` робить резервну копію лише даних v3. Для даних v2 дивіться [резервне копіювання сховища v2](/docs/v2.3/admin_guide#backing-up-the-datastore).

#### Змішані версії {#mixed-versions}

Під час оновлення кластер etcd підтримує змішані версії учасників etcd і працює з протоколом найнижчої загальної версії. Кластер вважається оновленим лише після того, як усі його учасники будуть оновлені до версії 3.2. Внутрішньо члени etcd домовляються між собою, щоб визначити загальну версію кластера, яка контролює версію, про яку повідомляється, та підтримувані функції.


#### Обмеження {#limitations}

Примітка: Якщо кластер містить лише дані v3 і не містить даних v2, це обмеження не застосовується.

Якщо кластер обслуговує набір даних v2 розміром понад 50 МБ, кожен новий оновлений учасник може зайняти до двох хвилин, щоб наздогнати поточний кластер. Перевірте розмір останнього знімка, щоб оцінити загальний розмір даних. Іншими словами, найкраще зачекати 2 хвилини між оновленням кожного учасника.

Для набагато більшого загального розміру даних, 100 МБ або більше, цей одноразовий процес може зайняти ще більше часу. Адміністратори дуже великих кластерів etcd такого масштабу можуть звернутися до [команди etcd][etcd-contact] перед оновленням, і ми будемо раді надати поради щодо процедури.

#### Пониження версії {#downgrade}

Якщо всі учасники були оновлені до версії 3.2, кластер буде оновлено до версії 3.2, і пониження з цього завершеного стану **неможливе**. Однак, якщо будь-який окремий учасник все ще є версією 3.1, кластер і його операції залишаються "v3.1", і з цього змішаного стану кластера можна повернутися до використання бінарного файлу etcd версії 3.1 на всіх учасниках.

Будь ласка, [зробіть резервну копію теки даних](../../op-guide/maintenance#snapshot-backup) всіх учасників etcd, щоб зробити можливим пониження версії кластера навіть після його повного оновлення.

### Процедура оновлення {#upgrade-procedure}

Цей приклад показує, як оновити кластер v3.1 etcd з 3 учасниками, що працює на локальній машині.

#### 1. Перевірте вимоги до оновлення {#check-upgrade-requirements}

Чи кластер справний і працює на версії v3.1.x?

```sh
$ ETCDCTL_API=3 etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:2379 is healthy: successfully committed proposal: took = 6.600684ms
localhost:22379 is healthy: successfully committed proposal: took = 8.540064ms
localhost:32379 is healthy: successfully committed proposal: took = 8.763432ms

$ curl http://localhost:2379/version
{"etcdserver":"3.1.7","etcdcluster":"3.1.0"}
```

#### 2. Зупиніть поточний процес etcd {#stop-the-existing-etcd-process}

Коли кожен процес etcd зупиняється, очікувані помилки будуть зареєстровані іншими учасниками кластера. Це нормально, оскільки зʼєднання учасника кластера було (тимчасово) розірвано:

```log
2017-04-27 14:13:31.491746 I | raft: c89feb932daef420 [term 3] received MsgTimeoutNow from 6d4f535bae3ab960 and starts an election to get leadership.
2017-04-27 14:13:31.491769 I | raft: c89feb932daef420 became candidate at term 4
2017-04-27 14:13:31.491788 I | raft: c89feb932daef420 received MsgVoteResp from c89feb932daef420 at term 4
2017-04-27 14:13:31.491797 I | raft: c89feb932daef420 [logterm: 3, index: 9] sent MsgVote request to 6d4f535bae3ab960 at term 4
2017-04-27 14:13:31.491805 I | raft: c89feb932daef420 [logterm: 3, index: 9] sent MsgVote request to 9eda174c7df8a033 at term 4
2017-04-27 14:13:31.491815 I | raft: raft.node: c89feb932daef420 lost leader 6d4f535bae3ab960 at term 4
2017-04-27 14:13:31.524084 I | raft: c89feb932daef420 received MsgVoteResp from 6d4f535bae3ab960 at term 4
2017-04-27 14:13:31.524108 I | raft: c89feb932daef420 [quorum:2] has received 2 MsgVoteResp votes and 0 vote rejections
2017-04-27 14:13:31.524123 I | raft: c89feb932daef420 became leader at term 4
2017-04-27 14:13:31.524136 I | raft: raft.node: c89feb932daef420 elected leader c89feb932daef420 at term 4
2017-04-27 14:13:31.592650 W | rafthttp: lost the TCP streaming connection with peer 6d4f535bae3ab960 (stream MsgApp v2 reader)
2017-04-27 14:13:31.592825 W | rafthttp: lost the TCP streaming connection with peer 6d4f535bae3ab960 (stream Message reader)
2017-04-27 14:13:31.693275 E | rafthttp: failed to dial 6d4f535bae3ab960 on stream Message (dial tcp [::1]:2380: getsockopt: connection refused)
2017-04-27 14:13:31.693289 I | rafthttp: peer 6d4f535bae3ab960 became inactive
2017-04-27 14:13:31.936678 W | rafthttp: lost the TCP streaming connection with peer 6d4f535bae3ab960 (stream Message writer)
```

На цьому етапі доцільно [зробити резервну копію даних etcd](../../op-guide/maintenance#snapshot-backup), щоб забезпечити шлях до пониження версії у разі виникнення будь-яких проблем:

```sh
$ etcdctl snapshot save backup.db
```

#### 3. Вставте бінарний файл etcd v3.2 і запустіть новий процес etcd {#drop-in-etcd-v3-2-binary-and-start-the-new-etcd-process}

Новий etcd v3.2 опублікує свою інформацію в кластер:

```log
2017-04-27 14:14:25.363225 I | etcdserver: published {Name:s1 ClientURLs:[http://localhost:2379]} to cluster a9ededbffcb1b1f1
```

Переконайтеся, що кожен учасник, а потім і весь кластер, стає справним з новим бінарним файлом etcd v3.2:

```sh
$ ETCDCTL_API=3 /etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:22379 is healthy: successfully committed proposal: took = 5.540129ms
localhost:32379 is healthy: successfully committed proposal: took = 7.321771ms
localhost:2379 is healthy: successfully committed proposal: took = 10.629901ms
```

Оновлені учасники будуть реєструвати попередження, подібні до наведених нижче, доки весь кластер не буде оновлено. Це очікувано і припиниться після того, як усі учасники кластера etcd будуть оновлені до версії 3.2:

```log
2017-04-27 14:15:17.071804 W | etcdserver: member c89feb932daef420 has a higher version 3.2.0
2017-04-27 14:15:21.073110 W | etcdserver: the local etcd version 3.1.7 is not up-to-date
2017-04-27 14:15:21.073142 W | etcdserver: member 6d4f535bae3ab960 has a higher version 3.2.0
2017-04-27 14:15:21.073157 W | etcdserver: the local etcd version 3.1.7 is not up-to-date
2017-04-27 14:15:21.073164 W | etcdserver: member c89feb932daef420 has a higher version 3.2.0
```

#### 4. Повторіть кроки 2 і 3 для всіх інших учасників {#repeat-step-2-to-step-3-for-all-other-members}

#### 5. Завершення {#finish}

Коли всі учасники будуть оновлені, кластер повідомить про успішне оновлення до версії 3.2:

```log
2017-04-27 14:15:54.536901 N | etcdserver/membership: updated the cluster version from 3.1 to 3.2
2017-04-27 14:15:54.537035 I | etcdserver/api: enabled capabilities for version 3.2
```

```sh
$ ETCDCTL_API=3 /etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:2379 is healthy: successfully committed proposal: took = 2.312897ms
localhost:22379 is healthy: successfully committed proposal: took = 2.553476ms
localhost:32379 is healthy: successfully committed proposal: took = 2.517902ms
```

[etcd-contact]: https://groups.google.com/g/etcd-dev

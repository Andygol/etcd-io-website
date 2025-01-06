---
title: Оновлення etcd з 3.2 до 3.3
weight: 6750
description: Процеси, контрольні списки та примітки щодо оновлення etcd з 3.2 до 3.3
---

У загальному випадку, оновлення з etcd 3.2 до 3.3 може бути безперервним, поступовим оновленням:

- по черзі зупиняйте процеси etcd v3.2 і замінюйте їх процесами etcd v3.3
- після запуску всіх процесів v3.3 нові функції v3.3 будуть доступні кластеру

Перед тим як [почати оновлення](#upgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки для оновлення {#upgrade-checklists}

**ПРИМІТКА:** При [міграції з v2 без даних v3](https://github.com/etcd-io/etcd/issues/9480), сервер etcd v3.2+ панікує, коли etcd відновлюється з наявних знімків, але немає файлу v3 `ETCD_DATA_DIR/member/snap/db`. Це відбувається, коли сервер мігрував з v2 без попередніх даних v3. Це також запобігає випадковій втраті даних v3 (наприклад, файл `db` міг бути переміщений). etcd вимагає, щоб після міграції v3 були тільки дані v3. Не оновлюйте до новіших версій v3, поки сервер v3.0 не містить даних v3.

**ПРИМІТКА:** якщо ви ввімкнули автентифікацію та використовуєте оренду (час життя оренди малий), існує висока ймовірність зіткнутися з [проблемою](https://github.com/etcd-io/etcd/issues/11689), яка призведе до неконсистентності даних. Наполегливо рекомендується спочатку оновитися до 3.2.31+, щоб розвʼязати цю проблему, а потім оновитися до 3.3. Крім того, якщо користувач без дозволу надсилає запит `LeaseRevoke` до вузла 3.3 під час процесу оновлення, це може призвести до пошкодження даних, тому краще переконатися, що у вашому середовищі немає таких аномальних викликів перед оновленням, див. [#11691](https://github.com/etcd-io/etcd/pull/11691) для деталей.

Основні зміни, що ламають сумісність у 3.3.

#### Змінено тип значення прапорця `etcd --auto-compaction-retention` на `string` {#changed-value-type-of-etcd-auto-compaction-retention-flag-to-string}

Змінено прапорця `--auto-compaction-retention` на [прийняття текстових значень](https://github.com/etcd-io/etcd/pull/8563) з [тоншою градацією](https://github.com/etcd-io/etcd/issues/8503). Тепер, коли `--auto-compaction-retention` приймає текстові значення, поле `auto-compaction-retention` у файлі конфігурації etcd YAML має бути змінено на тип `string`. Раніше, `--config-file etcd.config.yaml` міг мати поле `auto-compaction-retention: 24`, тепер має бути `auto-compaction-retention: "24"` або `auto-compaction-retention: "24h"`. Якщо налаштовано як `--auto-compaction-mode periodic --auto-compaction-retention "24h"`, значення тривалості часу для прапорця `--auto-compaction-retention` має бути дійсним для функції [`time.ParseDuration`](https://golang.org/pkg/time/#ParseDuration) у Go.

```diff
# etcd.config.yaml
+auto-compaction-mode: periodic
-auto-compaction-retention: 24
+auto-compaction-retention: "24"
+# Або
+auto-compaction-retention: "24h"
```

#### Змінено `etcdserver.EtcdServer.ServerConfig` на `*etcdserver.EtcdServer.ServerConfig` {#changed-etcdserver-etcdserver-serverconfig-to-etcdserver-etcdserver-serverconfig}

`etcdserver.EtcdServer` змінив тип свого поля-члена `*etcdserver.ServerConfig` на `etcdserver.ServerConfig`. І `etcdserver.NewServer` тепер приймає `etcdserver.ServerConfig`, замість `*etcdserver.ServerConfig`.

До і після (наприклад, [k8s.io/kubernetes/test/e2e_node/services/etcd.go](https://github.com/kubernetes/kubernetes/blob/release-1.8/test/e2e_node/services/etcd.go#L50-L55))

```diff
import "github.com/coreos/etcd/etcdserver"

type EtcdServer struct {
	*etcdserver.EtcdServer
-	config *etcdserver.ServerConfig
+	config etcdserver.ServerConfig
}

func NewEtcd(dataDir string) *EtcdServer {
-	config := &etcdserver.ServerConfig{
+	config := etcdserver.ServerConfig{
		DataDir: dataDir,
        ...
	}
	return &EtcdServer{config: config}
}

func (e *EtcdServer) Start() error {
	var err error
	e.EtcdServer, err = etcdserver.NewServer(e.config)
    ...
```

#### Додано структуру `embed.Config.LogOutput` {#added-embed-config-logoutput-struct}

**Зверніть увагу, що це поле було перейменовано на `embed.Config.LogOutputs` у типі `[]string` у v3.4. Будь ласка, дивіться [посібник з оновлення до v3.4](/docs/{{< param version >}}/upgrades/upgrade_3_4/) для отримання додаткової інформації.**

Поле `LogOutput` додано до `embed.Config`:

```diff
package embed

type Config struct {
 	Debug bool `json:"debug"`
 	LogPkgLevels string `json:"log-package-levels"`
+	LogOutput string `json:"log-output"`
 	...
```

Раніше попередження сервера gRPC записувалися в etcdserver.

```log
WARNING: 2017/11/02 11:35:51 grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: Error while dialing dial tcp: operation was canceled"; Reconnecting to {localhost:2379 <nil>}
WARNING: 2017/11/02 11:35:51 grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: Error while dialing dial tcp: operation was canceled"; Reconnecting to {localhost:2379 <nil>}
```

З версії v3.3, журнали сервера gRPC стандартно вимкнені.

**Зверніть увагу, що метод `embed.Config.SetupLogging` був застарілий у v3.4. Будь ласка, дивіться [посібник з оновлення до v3.4](/docs/{{< param version >}}/upgrades/upgrade_3_4/) для отримання додаткової інформації.**

```go
import "github.com/coreos/etcd/embed"

cfg := &embed.Config{Debug: false}
cfg.SetupLogging()
```

Встановіть поле `embed.Config.Debug` у `true`, щоб увімкнути журнали сервера gRPC.

#### Змінено відповідь точки доступу `/health` {#changed-health-endpoint-response}

Раніше, `[endpoint]:[client-port]/health` повертав вручну серіалізоване значення JSON. 3.3 тепер визначає структуру [`etcdhttp.Health`](https://pkg.go.dev/github.com/etcd-io/etcd/etcdserver/api/etcdhttp#Health).

Зверніть увагу, що у версіях v3.3.0-rc.0, v3.3.0-rc.1 та v3.3.0-rc.2, `etcdhttp.Health` має поля типу boolean `"health"` та `"errors"`. Для зворотної сумісності ми повернули поле `"health"` до типу `string` і видалили поле `"errors"`. Додаткова інформація про здоров'я буде надана в окремих API.

```bash
$ curl http://localhost:2379/health
{"health":"true"}
```

#### Змінено HTTP кінцеві точки gRPC шлюзу (замінено `/v3alpha` на `/v3beta`) {#changed-grpc-gateway-http-endpoints-replaced-v3alpha-with-v3beta}

До

```bash
curl -L http://localhost:2379/v3alpha/kv/put \
  -X POST -d '{"key": "Zm9v", "value": "YmFy"}'
```

Після

```bash
curl -L http://localhost:2379/v3beta/kv/put \
  -X POST -d '{"key": "Zm9v", "value": "YmFy"}'
```

Запити до точок доступу `/v3alpha` будуть перенаправлені на `/v3beta`, а `/v3alpha` буде видалено у випуску 3.4.

#### Змінено максимальні обмеження розміру запиту {#changed-maximum-request-size-limits}

3.3 тепер дозволяє налаштовувати обмеження розміру запиту як для сервера, так і для **клієнтської сторони**. У попередніх версіях (v3.2.10, v3.2.11) розмір відповіді клієнта був обмежений лише 4 МіБ.

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

// обмежити запити до 5 МіБ
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

#### Змінено сигнатури функцій обгортки клієнта gRPC clientv3 {#changed-raw-grpc-client-wrapper-function-signatures}

3.3 змінює сигнатури функцій обгортки клієнта gRPC `clientv3`. Ця зміна була необхідна для підтримки [власного `grpc.CallOption` на обмеження розміру повідомлення](https://github.com/etcd-io/etcd/pull/9047).

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

#### Змінено тип помилки API `Snapshot` clientv3 {#changed-clientv3-snapshot-api-error-type}

Раніше, API `Snapshot` clientv3 повертав необроблений тип помилки [`grpc/*status.statusError`]. У версії 3.3 ці помилки тепер переводяться у відповідні загальнодоступні типи помилок, щоб бути сумісними з іншими API.

До

```go
import "context"

// читання знімка з скасованим контекстом має викликати помилку
ctx, cancel := context.WithCancel(context.Background())
rc, _ := cli.Snapshot(ctx)
cancel()
_, err := io.Copy(f, rc)
err.Error() == "rpc error: code = Canceled desc = context canceled"

// читання знімка з перевищеним терміном має викликати помилку
ctx, cancel = context.WithTimeout(context.Background(), time.Second)
defer cancel()
rc, _ = cli.Snapshot(ctx)
time.Sleep(2 * time.Second)
_, err = io.Copy(f, rc)
err.Error() == "rpc error: code = DeadlineExceeded desc = context deadline exceeded"
```

Після

```go
import "context"

// читання знімка з скасованим контекстом має викликати помилку
ctx, cancel := context.WithCancel(context.Background())
rc, _ := cli.Snapshot(ctx)
cancel()
_, err := io.Copy(f, rc)
err == context.Canceled

// читання знімка з перевищеним терміном має викликати помилку
ctx, cancel = context.WithTimeout(context.Background(), time.Second)
defer cancel()
rc, _ = cli.Snapshot(ctx)
time.Sleep(2 * time.Second)
_, err = io.Copy(f, rc)
err == context.DeadlineExceeded
```

#### Змінено вивід команди `etcdctl lease timetolive` {#changed-etcdctl-lease-timetolive-command-output}

Раніше, команда `lease timetolive LEASE_ID` на закінченій оренді виводила `-1s` для залишкових секунд. 3.3 тепер виводить більш зрозумілі повідомлення.

До

```bash
lease 2d8257079fa1bc0c granted with TTL(0s), remaining(-1s)
```

Після

```bash
lease 2d8257079fa1bc0c already expired
```

#### Змінено імпорти `golang.org/x/net/context` {#changed-golang-org-x-net-context-imports}

`clientv3` застарів `golang.org/x/net/context`. Якщо проєкт використовує `golang.org/x/net/context` в іншому коді (наприклад, згенерованому протокольному буферному коді etcd) та імпортує `github.com/coreos/etcd/clientv3`, для компіляції потрібна Go 1.9+.

До

```go
import "golang.org/x/net/context"
cli.Put(context.Background(), "f", "v")
```

Після

```go
import "context"
cli.Put(context.Background(), "f", "v")
```

#### Змінено залежність gRPC {#changed-grpc-dependency}

3.3 тепер вимагає [grpc/grpc-go](https://github.com/grpc/grpc-go/releases) `v1.7.5`.

##### Застарілий `grpclog.Logger` {#deprecated-grpclog-logger}

`grpclog.Logger` застарів на користь [`grpclog.LoggerV2`](https://github.com/grpc/grpc-go/blob/master/grpclog/loggerv2.go). `clientv3.Logger` тепер `grpclog.LoggerV2`.

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

// log.New вище не можна використовувати (не реалізує інтерфейс grpclog.LoggerV2)
```

##### Застарілий `grpc.ErrClientConnTimeout` {#deprecated-grpc-errclientconntimeout}

Раніше, помилка `grpc.ErrClientConnTimeout` поверталася при тайм-аутах підключення клієнта. 3.3 замість цього повертає `context.DeadlineExceeded` (див. [#8504](https://github.com/etcd-io/etcd/issues/8504)).

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

#### Змінено офіційний реєстр контейнерів {#changed-official-container-registry}

etcd тепер використовує [`gcr.io/etcd-development/etcd`](https://gcr.io/etcd-development/etcd) як основний реєстр контейнерів, і [`quay.io/coreos/etcd`](https://quay.io/coreos/etcd) як вторинний.

До

```bash
docker pull quay.io/coreos/etcd:v3.2.5
```

Після

```bash
docker pull gcr.io/etcd-development/etcd:v3.3.0
```

### Оновлення до >= v3.3.14 {#upgrades-to-v3-3-14}

У [v3.3.14](https://github.com/etcd-io/etcd/releases/tag/v3.3.14) довелося включити деякі функції з 3.4, намагаючись мінімізувати різницю між реалізацією балансувальника клієнтів. Цей випуск виправляє ["kube-apiserver 1.13.x відмовляється працювати, коли перший etcd-сервер недоступний" (kubernetes#72102)](https://github.com/kubernetes/kubernetes/issues/72102).

`grpc.ErrClientConnClosing` було визнано [застарілим у gRPC >= 1.10](https://github.com/grpc/grpc-go/pull/1854).

```diff
import (
+	"go.etcd.io/etcd/clientv3"

	"google.golang.org/grpc"
+	"google.golang.org/grpc/codes"
+	"google.golang.org/grpc/status"
)

_, err := kvc.Get(ctx, "a")
-if err == grpc.ErrClientConnClosing {
+if clientv3.IsConnCanceled(err) {

// or
+s, ok := status.FromError(err)
+if ok {
+  if s.Code() == codes.Canceled
```

[Новий балансувальник клієнтів](/docs/{{< param version >}}/learning/design-client/) використовує асинхронний резолвер для передачі точок доступу у функцію підключення gRPC. Як результат, [v3.3.14](https://github.com/etcd-io/etcd/releases/tag/v3.3.14) або пізніша версія вимагає опцію підключення `grpc.WithBlock`, щоб чекати, поки підключення не буде встановлено.

```diff
import (
	"time"
	"go.etcd.io/etcd/clientv3"
+	"google.golang.org/grpc"
)

+// "grpc.WithBlock()" для блокування до встановлення підключення
ccfg := clientv3.Config{
  Endpoints:            []string{"localhost:2379"},
  DialTimeout:          time.Second,
+ DialOptions:          []grpc.DialOption{grpc.WithBlock()},
  DialKeepAliveTime:    time.Second,
  DialKeepAliveTimeout: 500 * time.Millisecond,
}
```

Будь ласка, дивіться [CHANGELOG](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.3.md) для повного списку змін.

### Контрольні списки для оновлення сервера {#server-upgrade-checklists}

#### Вимоги до оновлення {#upgrade-requirements}

Щоб оновити наявне розгортання etcd до 3.3, кластер, що працює, повинен бути версії 3.2 або вище. Якщо версія нижче 3.2, будь ласка, [оновіть до 3.2](/docs/v3.5/upgrades/upgrade_3_2/) перед оновленням до 3.3.

Також, щоб забезпечити плавне поступове оновлення, кластер, що працює, повинен бути справним. Перевірте справність кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед оновленням etcd завжди тестуйте сервіси, що залежать від etcd, у тестовому середовищі перед розгортанням оновлення у промисловому середовищі.

Перед початком, [зробіть резервну копію даних etcd](../../op-guide/maintenance/#snapshot-backup). Якщо щось піде не так під час оновлення, можна використовувати цю резервну копію для [пониження версії](#downgrade) до поточної версії etcd. Зверніть увагу, що команда `snapshot` робить резервну копію лише даних v3. Для даних v2 див. [резервне копіювання сховища v2](/docs/v2.3/admin_guide/#backing-up-the-datastore).

#### Змішані версії {#mixed-versions}

Під час оновлення кластер etcd підтримує змішані версії членів etcd і працює з протоколом найнижчої загальної версії. Кластер вважається оновленим лише після того, як усі його члени будуть оновлені до версії 3.3. Внутрішньо члени etcd домовляються між собою, щоб визначити загальну версію кластера, яка контролює версію, про яку повідомляється, та підтримувані функції.

#### Обмеження {#limitations}

Примітка: Якщо кластер має лише дані v3 і не має даних v2, це обмеження не застосовується.

Якщо кластер обслуговує набір даних v2 розміром понад 50 МБ, кожен новий оновлений член може зайняти до двох хвилин, щоб наздогнати поточний кластер. Перевірте розмір останнього знімка, щоб оцінити загальний розмір даних. Іншими словами, найкраще чекати 2 хвилини між оновленням кожного члена.

Для набагато більшого загального розміру даних, 100 МБ або більше, цей одноразовий процес може зайняти ще більше часу. Адміністратори дуже великих кластерів etcd такого масштабу можуть звернутися до [команди etcd][etcd-contact] перед оновленням, і ми будемо раді надати поради щодо процедури.

#### Пониження версії {#downgrade}

Якщо всі члени були оновлені до v3.3, кластер буде оновлено до v3.3, і пониження версії з цього завершеного стану **неможливе**. Однак, якщо будь-який окремий член все ще є v3.2, кластер і його операції залишаються "v3.2", і з цього змішаного стану кластера можна повернутися до використання бінарного файлу etcd v3.2 на всіх членах.

Будь ласка, [зробіть резервну копію теки даних](../../op-guide/maintenance/#snapshot-backup) всіх членів etcd, щоб зробити можливим пониження версії кластера навіть після його повного оновлення.

### Процедура оновлення {#upgrade-procedure}

Цей приклад показує, як оновити кластер v3.2 з 3 членами, що працює на локальній машині.

#### 1. Перевірте вимоги до оновлення {#check-upgrade-requirements}

Чи кластер справний і працює на v3.2.x?

```sh
$ ETCDCTL_API=3 etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:2379 is healthy: successfully committed proposal: took = 6.600684ms
localhost:22379 is healthy: successfully committed proposal: took = 8.540064ms
localhost:32379 is healthy: successfully committed proposal: took = 8.763432ms

$ curl http://localhost:2379/version
{"etcdserver":"3.2.7","etcdcluster":"3.2.0"}
```

#### 2. Зупиніть поточний процес etcd {#stop-the-existing-etcd-process}

Коли кожен процес etcd зупиняється, очікувані помилки будуть записані іншими членами кластера. Це нормально, оскільки зʼєднання члена кластера було (тимчасово) розірвано:

```log
14:13:31.491746 I | raft: c89feb932daef420 [term 3] received MsgTimeoutNow from 6d4f535bae3ab960 and starts an election to get leadership.
14:13:31.491769 I | raft: c89feb932daef420 became candidate at term 4
14:13:31.491788 I | raft: c89feb932daef420 received MsgVoteResp from c89feb932daef420 at term 4
14:13:31.491797 I | raft: c89feb932daef420 [logterm: 3, index: 9] sent MsgVote request to 6d4f535bae3ab960 at term 4
14:13:31.491805 I | raft: c89feb932daef420 [logterm: 3, index: 9] sent MsgVote request to 9eda174c7df8a033 at term 4
14:13:31.491815 I | raft: raft.node: c89feb932daef420 lost leader 6d4f535bae3ab960 at term 4
14:13:31.524084 I | raft: c89feb932daef420 received MsgVoteResp from 6d4f535bae3ab960 at term 4
14:13:31.524108 I | raft: c89feb932daef420 [quorum:2] has received 2 MsgVoteResp votes and 0 vote rejections
14:13:31.524123 I | raft: c89feb932daef420 became leader at term 4
14:13:31.524136 I | raft: raft.node: c89feb932daef420 elected leader c89feb932daef420 at term 4
14:13:31.592650 W | rafthttp: lost the TCP streaming connection with peer 6d4f535bae3ab960 (stream MsgApp v2 reader)
14:13:31.592825 W | rafthttp: lost the TCP streaming connection with peer 6d4f535bae3ab960 (stream Message reader)
14:13:31.693275 E | rafthttp: failed to dial 6d4f535bae3ab960 on stream Message (dial tcp [::1]:2380: getsockopt: connection refused)
14:13:31.693289 I | rafthttp: peer 6d4f535bae3ab960 became inactive
14:13:31.936678 W | rafthttp: lost the TCP streaming connection with peer 6d4f535bae3ab960 (stream Message writer)
```

На цьому етапі добре [зробити резервну копію даних etcd](../../op-guide/maintenance/#snapshot-backup), щоб забезпечити шлях до пониження версії у разі виникнення будь-яких проблем:

```sh
$ etcdctl snapshot save backup.db
```

#### 3. Вставте бінарний файл etcd v3.3 і запустіть новий процес etcd {#drop-in-etcd-v3-3-binary-and-start-the-new-etcd-process}

Новий etcd v3.3 опублікує свою інформацію в кластер:

```log
14:14:25.363225 I | etcdserver: published {Name:s1 ClientURLs:[http://localhost:2379]} to cluster a9ededbffcb1b1f1
```

Переконайтеся, що кожен член, а потім весь кластер, стає здоровим з новим бінарним файлом etcd v3.3:

```sh
$ ETCDCTL_API=3 /etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:22379 is healthy: successfully committed proposal: took = 5.540129ms
localhost:32379 is healthy: successfully committed proposal: took = 7.321771ms
localhost:2379 is healthy: successfully committed proposal: took = 10.629901ms
```

Оновлені члени будуть записувати попередження, як наведені нижче, поки весь кластер не буде оновлено. Це очікувано і припиниться після того, як усі члени кластера etcd будуть оновлені до v3.3:

```log
14:15:17.071804 W | etcdserver: member c89feb932daef420 has a higher version 3.3.0
14:15:21.073110 W | etcdserver: the local etcd version 3.2.7 is not up-to-date
14:15:21.073142 W | etcdserver: member 6d4f535bae3ab960 has a higher version 3.3.0
14:15:21.073157 W | etcdserver: the local etcd version 3.2.7 is not up-to-date
14:15:21.073164 W | etcdserver: member c89feb932daef420 has a higher version 3.3.0
```

#### 4. Повторіть крок 2 до кроку 3 для всіх інших членів {#repeat-step-2-to-step-3-for-all-other-members}

#### 5. Завершення {#finish}

Коли всі члени будуть оновлені, кластер повідомить про успішне оновлення до 3.3:

```log
14:15:54.536901 N | etcdserver/membership: updated the cluster version from 3.2 to 3.3
14:15:54.537035 I | etcdserver/api: enabled capabilities for version 3.3
```

```sh
$ ETCDCTL_API=3 /etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
localhost:2379 is healthy: successfully committed proposal: took = 2.312897ms
localhost:22379 is healthy: successfully committed proposal: took = 2.553476ms
localhost:32379 is healthy: successfully committed proposal: took = 2.517902ms
```

[etcd-contact]: https://groups.google.com/g/etcd-dev

---
title: Оновлення etcd з 3.3 до 3.4
weight: 6700
description: Процеси, контрольні списки та примітки щодо оновлення etcd з 3.3 до 3.4
---

У загальному випадку, оновлення з etcd 3.3 до 3.4 може бути безперервним, поступовим оновленням:

- по черзі зупиняйте процеси etcd v3.3 і замінюйте їх процесами etcd v3.4
- після запуску всіх процесів v3.4 нові функції v3.4 будуть доступні кластеру

Перед [початком оновлення](#upgrade-procedure), прочитайте решту цього посібника, щоб підготуватися.

### Контрольні списки оновлення {#upgrade-checklists}

**ПРИМІТКА:** При [міграції з v2 без даних v3](https://github.com/etcd-io/etcd/issues/9480), сервер etcd v3.2+ панікує, коли etcd відновлюється з наявних знімків, але немає файлу v3 `ETCD_DATA_DIR/member/snap/db`. Це відбувається, коли сервер мігрував з v2 без попередніх даних v3. Це також запобігає випадковій втраті даних v3 (наприклад, файл `db` міг бути переміщений). etcd вимагає, щоб після міграції v3 могли відбуватися лише з даними v3. Не оновлюйтеся до новіших версій v3, доки сервер v3.0 не містить даних v3.

Основні зміни в 3.4.

#### Зробити `ETCDCTL_API=3 etcdctl` стандартним {#make-etcdctl_api-3-etcdctl-default}

`ETCDCTL_API=3` тепер є стандартним.

```diff
etcdctl set foo bar
Error: unknown command "set" for "etcdctl"

-etcdctl set foo bar
+ETCDCTL_API=2 etcdctl set foo bar
bar

ETCDCTL_API=3 etcdctl put foo bar
OK

-ETCDCTL_API=3 etcdctl put foo bar
+etcdctl put foo bar
```

#### Зробити `etcd --enable-v2=false` стандартним {#make-etcd-enable-v2-false-default}

[`etcd --enable-v2=false`](https://github.com/etcd-io/etcd/pull/10935) тепер є стандартним.

Це означає, що якщо не вказано `etcd --enable-v2=true`, сервер etcd v3.4 не обслуговуватиме запити API v2.

Якщо використовувався API v2, переконайтеся, що увімкнено API v2 у v3.4:

```diff
-etcd
+etcd --enable-v2=true
```

Інші HTTP API все ще працюватимуть (наприклад, `[CLIENT-URL]/metrics`, `[CLIENT-URL]/health`, v3 gRPC gateway).

#### Застарілі прапорці `etcd --ca-file` та `etcd --peer-ca-file` {#deprecated-etcd-ca-file-and-etcd-peer-ca-file-flags}

Прапорці `--ca-file` та `--peer-ca-file` застарілі; вони застаріли з v2.1.

Зазначення цього параметра також автоматично ввімкне автентифікацію клієнтських сертифікатів незалежно від значення, встановленого для `--client-cert-auth`.

```diff
-etcd --ca-file ca-client.crt
+etcd --trusted-ca-file ca-client.crt
```

```diff
-etcd --peer-ca-file ca-peer.crt
+etcd --peer-trusted-ca-file ca-peer.crt
```

#### Застаріла помилка `grpc.ErrClientConnClosing` {#deprecated-grpc-errclientconnclosing-error}

`grpc.ErrClientConnClosing` була [застаріла в gRPC >= 1.10](https://github.com/grpc/grpc-go/pull/1854).

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

#### Вимагати `grpc.WithBlock` для клієнтського підключення {#require-grpc-withblock-for-client-dial}

[Новий клієнтський балансувальник](/docs/{{< param version >}}/learning/design-client/) використовує асинхронний резолвер для передачі точок доступу до функції підключення gRPC. В результаті клієнт v3.4 вимагає опцію підключення `grpc.WithBlock`, щоб чекати, поки підключення не буде встановлено.

```diff
import (
	"time"
	"go.etcd.io/etcd/clientv3"
+	"google.golang.org/grpc"
)

+// "grpc.WithBlock()" щоб блокувати, поки підключення не буде встановлено
ccfg := clientv3.Config{
  Endpoints:            []string{"localhost:2379"},
  DialTimeout:          time.Second,
+ DialOptions:          []grpc.DialOption{grpc.WithBlock()},
  DialKeepAliveTime:    time.Second,
  DialKeepAliveTimeout: 500 * time.Millisecond,
}
```

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_db_total_size_in_bytes` {#deprecating-etcd_debugging_mvcc_db_total_size_in_bytes-prometheus-metrics}

v3.4 підвищує метрики Prometheus `etcd_debugging_mvcc_db_total_size_in_bytes` до `etcd_mvcc_db_total_size_in_bytes`, щоб заохотити моніторинг сховища etcd.

`etcd_debugging_mvcc_db_total_size_in_bytes` все ще обслуговується у v3.4 для зворотної сумісності. Він буде повністю застарілий у v3.5.

```diff
-etcd_debugging_mvcc_db_total_size_in_bytes
+etcd_mvcc_db_total_size_in_bytes
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_put_total` {#deprecating-etcd_debugging_mvcc_put_total-prometheus-metrics}

v3.4 підвищує метрики Prometheus `etcd_debugging_mvcc_put_total` до `etcd_mvcc_put_total`, щоб заохотити моніторинг сховища etcd.

`etcd_debugging_mvcc_put_total` все ще обслуговується у v3.4 для зворотної сумісності. Він буде повністю застарілий у v3.5.

```diff
-etcd_debugging_mvcc_put_total
+etcd_mvcc_put_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_delete_total` {#deprecating-etcd_debugging_mvcc_delete_total-prometheus-metrics}

v3.4 підвищує метрики Prometheus `etcd_debugging_mvcc_delete_total` до `etcd_mvcc_delete_total`, щоб заохотити моніторинг сховища etcd.

`etcd_debugging_mvcc_delete_total` все ще обслуговується у v3.4 для зворотної сумісності. Він буде повністю застарілий у v3.5.

```diff
-etcd_debugging_mvcc_delete_total
+etcd_mvcc_delete_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_txn_total` {#deprecating-etcd_debugging_mvcc_txn_total-prometheus-metrics}

v3.4 підвищує метрики Prometheus `etcd_debugging_mvcc_txn_total` до `etcd_mvcc_txn_total`, щоб заохотити моніторинг сховища etcd.

`etcd_debugging_mvcc_txn_total` все ще обслуговується у v3.4 для зворотної сумісності. Він буде повністю застарілий у v3.5.

```diff
-etcd_debugging_mvcc_txn_total
+etcd_mvcc_txn_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілі метрики Prometheus `etcd_debugging_mvcc_range_total` {#deprecating-etcd_debugging_mvcc_range_total-prometheus-metrics}

v3.4 підвищує метрики Prometheus `etcd_debugging_mvcc_range_total` до `etcd_mvcc_range_total`, щоб заохотити моніторинг сховища etcd.

`etcd_debugging_mvcc_range_total` все ще обслуговується у v3.4 для зворотної сумісності. Він буде повністю застарілий у v3.5.

```diff
-etcd_debugging_mvcc_range_total
+etcd_mvcc_range_total
```

Зверніть увагу, що метрики простору імен `etcd_debugging_*` були позначені як експериментальні. У міру покращення керівництва з моніторингу ми можемо підвищити більше метрик.

#### Застарілий прапорець `etcd --log-output` (тепер `--log-outputs`) {#deprecating-etcd-log-output-flag-now-log-outputs}

Перейменуйте [`etcd --log-output` на `--log-outputs`](https://github.com/etcd-io/etcd/pull/9624), щоб підтримувати кілька виводів журналу. **`etcd --logger=capnslog` не підтримує кілька виводів журналу.**

**`etcd --log-output`** буде застарілий у v3.5. **`etcd --logger=capnslog` буде застарілий у v3.5**.

```diff
-etcd --log-output=stderr
+etcd --log-outputs=stderr

+# щоб записувати журнали в stderr і файл a.log одночасно
+# лише "--logger=zap" підтримує кілька записувачів
+etcd --logger=zap --log-outputs=stderr,a.log
```

v3.4 додає підтримку `etcd --logger=zap --log-outputs=stderr` для структурованого журналювання та кількох виводів журналу. Основна мотивація полягає в тому, щоб сприяти автоматизованому моніторингу etcd, а не переглядати журнали сервера, коли він починає ламатися. Майбутній розвиток зробить журнали etcd якомога меншими та полегшить моніторинг etcd за допомогою метрик та сповіщень. **`etcd --logger=capnslog` буде застарілий у v3.5**.

#### Змінено тип поля `log-outputs` у `etcd --config-file` на `[]string` {#changed-log-outputs-field-type-in-etcd-config-file-to-string}

Тепер, коли `log-outputs` (старе імʼя поля `log-output`) приймає кілька записувачів, поле `log-outputs` у конфігураційному файлі YAML etcd має бути змінено на тип `[]string`, як показано нижче:

```diff
 # Вкажіть 'stdout' або 'stderr', щоб пропустити журналювання journald, навіть коли працюєте під systemd.
-log-output: default
+log-outputs: [default]
```

#### Перейменовано `embed.Config.LogOutput` на `embed.Config.LogOutputs` {#renamed-embed-config-logoutput-to-embed-config-logoutputs}

Перейменовано [**`embed.Config.LogOutput`** на **`embed.Config.LogOutputs`**](https://github.com/etcd-io/etcd/pull/9624), щоб підтримувати кілька виводів журналу. І змінено [тип `embed.Config.LogOutput` з `string` на `[]string`](https://github.com/etcd-io/etcd/pull/9579), щоб підтримувати кілька виводів журналу.

```diff
import "github.com/coreos/etcd/embed"

cfg := &embed.Config{Debug: false}
-cfg.LogOutput = "stderr"
+cfg.LogOutputs = []string{"stderr"}
```

#### v3.5 визнає застарілим `capnslog` {#v3-5-deprecates-capnslog}

**v3.5 визнає застарілим прапорець `etcd --log-package-levels` для `capnslog`**; `etcd --logger=zap --log-outputs=stderr` стандартно буде. **v3.5 визнає застарілою точку доступу `[CLIENT-URL]/config/local/log`.**

```diff
-etcd
+etcd --logger zap
```

#### Застарілий прапорець `etcd --debug` (тепер `--log-level=debug`) {#deprecating-etcd-debug-flag-now-log-level-debug}

v3.4 визнає застарілим [прапорець `etcd --debug`](https://github.com/etcd-io/etcd/pull/10947). Натомість використовуйте прапорець `etcd --log-level=debug`.

```diff
-etcd --debug
+etcd --logger zap --log-level debug
```

#### Застаріле поле `pkg/transport.TLSInfo.CAFile` {#deprecated-pkg-transport-tlsinfo-cafile-field}

Застаріле поле `pkg/transport.TLSInfo.CAFile`.

```diff
import "github.com/coreos/etcd/pkg/transport"

tlsInfo := transport.TLSInfo{
    CertFile: "/tmp/test-certs/test.pem",
    KeyFile: "/tmp/test-certs/test-key.pem",
-   CAFile: "/tmp/test-certs/trusted-ca.pem",
+   TrustedCAFile: "/tmp/test-certs/trusted-ca.pem",
}
tlsConfig, err := tlsInfo.ClientConfig()
if err != nil {
    panic(err)
}
```

#### Змінено `embed.Config.SnapCount` на `embed.Config.SnapshotCount` {#changed-embed-config-snapcount-to-embed-config-snapshotcount}

Щоб бути послідовним з імʼям прапорця `etcd --snapshot-count`, поле `embed.Config.SnapCount` було перейменовано на `embed.Config.SnapshotCount`:

```diff
import "github.com/coreos/etcd/embed"

cfg := embed.NewConfig()
-cfg.SnapCount = 100000
+cfg.SnapshotCount = 100000
```

#### Змінено `etcdserver.ServerConfig.SnapCount` на `etcdserver.ServerConfig.SnapshotCount` {#changed-etcdserver-serverconfig-snapcount-to-etcdserver-serverconfig-snapshotcount}

Щоб бути послідовним з імʼям прапорця `etcd --snapshot-count`, поле `etcdserver.ServerConfig.SnapCount` було перейменовано на `etcdserver.ServerConfig.SnapshotCount`:

```diff
import "github.com/coreos/etcd/etcdserver"

srvcfg := etcdserver.ServerConfig{
-  SnapCount: 100000,
+  SnapshotCount: 100000,
```

#### Змінено сигнатуру функції в пакунку `wal` {#changed-function-signature-in-package-wal}

Змінено сигнатури функцій `wal`, щоб підтримувати структурований журнал.

```diff
import "github.com/coreos/etcd/wal"
+import "go.uber.org/zap"

+lg, _ = zap.NewProduction()

-wal.Open(dirpath, snap)
+wal.Open(lg, dirpath, snap)

-wal.OpenForRead(dirpath, snap)
+wal.OpenForRead(lg, dirpath, snap)

-wal.Repair(dirpath)
+wal.Repair(lg, dirpath)

-wal.Create(dirpath, metadata)
+wal.Create(lg, dirpath, metadata)
```

#### Змінено тип `IntervalTree` у пакеті `pkg/adt` {#changed-intervaltree-type-in-package-pkg-adt}

`pkg/adt.IntervalTree` тепер визначено як `інтерфейс`.

```diff
import (
    "fmt"

    "go.etcd.io/etcd/pkg/adt"
)

func main() {
-    ivt := &adt.IntervalTree{}
+    ivt := adt.NewIntervalTree()
```

#### Застаріле `embed.Config.SetupLogging` {#deprecated-embed-config-setuplogging}

`embed.Config.SetupLogging` було видалено, щоб запобігти неправильній конфігурації журналювання, і тепер налаштовується автоматично.

```diff
import "github.com/coreos/etcd/embed"

cfg := &embed.Config{Debug: false}
-cfg.SetupLogging()
```

#### Змінено HTTP точки доступу gRPC шлюзу (замість `/v3beta` використовується `/v3`) {#changed-grpc-gateway-http-endpoints-replaced-v3beta-with-v3}

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

Запити до точок доступу `/v3beta` будуть перенаправлені на `/v3`, а `/v3beta` буде видалено у випуску 3.5.

#### Застарілі теґи образів контейнерів {#deprecated-container-image-tags}

`latest` і теґи образів мінорних версій застарілі:

```diff
-docker pull gcr.io/etcd-development/etcd:latest
+docker pull gcr.io/etcd-development/etcd:v3.4.0

-docker pull gcr.io/etcd-development/etcd:v3.4
+docker pull gcr.io/etcd-development/etcd:v3.4.0

-docker pull gcr.io/etcd-development/etcd:v3.4
+docker pull gcr.io/etcd-development/etcd:v3.4.1

-docker pull gcr.io/etcd-development/etcd:v3.4
+docker pull gcr.io/etcd-development/etcd:v3.4.2
```

### Контрольні списки оновлення сервера {#server-upgrade-checklists}

#### Вимоги до оновлення {#upgrade-requirements}

Щоб оновити поточне розгортання etcd до 3.4, кластер, що працює, повинен бути 3.3 або вище. Якщо це до 3.3, будь ласка, [оновіться до 3.3](docs/v3.5/upgrades/upgrade_3_3/) перед оновленням до 3.4.

Крім того, щоб забезпечити плавне поступове оновлення, кластер, що працює, повинен бути справним. Перевірте стан кластера за допомогою команди `etcdctl endpoint health` перед продовженням.

#### Підготовка {#preparation}

Перед оновленням etcd завжди тестуйте служби, що залежать від etcd, у середовищі тестування перед розгортанням оновлення в робочому середовищі.

Перед початком [завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup). Якщо щось піде не так з оновленням, можна використовувати цю резервну копію для [пониження версії](#downgrade) до поточної версії etcd. Зверніть увагу, що команда `snapshot` створює резервну копію лише даних v3. Для даних v2 див. [резервне копіювання сховища v2](/docs/v2.3/admin_guide/#backing-up-the-datastore).

#### Змішані версії {#mixed-versions}

Під час оновлення кластер etcd підтримує змішані версії учасників etcd і працює з протоколом найнижчої загальної версії. Кластер вважається оновленим лише після того, як усі його учасники будуть оновлені до версії 3.4. Внутрішньо члени etcd домовляються між собою, щоб визначити загальну версію кластера, яка контролює версію, про яку повідомляється, та підтримувані функції.

#### Обмеження {#limitations}

Примітка: якщо кластер містить лише дані v3 і не містить даних v2, це обмеження не застосовується.

Якщо кластер обслуговує набір даних v2 розміром понад 50 МБ, кожному новому оновленому учаснику може знадобитися до двох хвилин, щоб наздогнати поточний кластер. Перевірте розмір останнього знімка, щоб оцінити загальний розмір даних. Іншими словами, найкраще почекати 2 хвилини між оновленням кожного учасника.

Для набагато більшого загального розміру даних, 100 МБ або більше, цей одноразовий процес може зайняти ще більше часу. Адміністратори дуже великих кластерів etcd такого масштабу можуть звернутися до [команди etcd][etcd-contact] перед оновленням, і ми будемо раді надати поради щодо процедури.

#### Пониження версії {#downgrade}

Якщо всі учасники були оновлені до v3.4, кластер буде оновлено до v3.4, і пониження версії з цього завершеного стану **неможливе**. Однак, якщо будь-який окремий учасник все ще є v3.3, кластер і його операції залишаються "v3.3", і з цього змішаного стану кластера можна повернутися до використання двійкового файлу etcd v3.3 на всіх учасниках.

Будь ласка, [завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup), щоб зробити можливим пониження версії кластера навіть після його повного оновлення.

### Процедура оновлення {#upgrade-procedure}

Цей приклад показує, як оновити кластер v3.3 з 3 учасниками, що працює на локальній машині.

#### Крок 1: перевірте вимоги до оновлення {#step-1-check-upgrade-requirements}

Чи кластер справний та працює на v3.3.x?

```bash
etcdctl --endpoints=localhost:2379,localhost:22379,localhost:32379 endpoint health
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 2.118638ms
localhost:22379 is healthy: successfully committed proposal: took = 3.631388ms
localhost:32379 is healthy: successfully committed proposal: took = 2.157051ms
COMMENT

curl http://localhost:2379/version
<<COMMENT
{"etcdserver":"3.3.5","etcdcluster":"3.3.0"}
COMMENT

curl http://localhost:22379/version
<<COMMENT
{"etcdserver":"3.3.5","etcdcluster":"3.3.0"}
COMMENT

curl http://localhost:32379/version
<<COMMENT
{"etcdserver":"3.3.5","etcdcluster":"3.3.0"}
COMMENT
```

#### Крок 2: завантажте резервну копію знімка з лідера {#step-2-download-snapshot-backup-from-leader}

[Завантажте резервну копію знімка](../../op-guide/maintenance/#snapshot-backup), щоб забезпечити шлях до пониження версії у разі виникнення будь-яких проблем.

лідер etcd гарантовано має найновіші дані програми, тому отримайте знімок від лідера:

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

Коли кожен процес etcd зупиняється, очікувані помилки будуть зареєстровані іншими учасниками кластера. Це нормально, оскільки зʼєднання учасника кластера було (тимчасово) розірвано:

```bash
10.237579 I | etcdserver: updating the cluster version from 3.0 to 3.3
10.238315 N | etcdserver/membership: updated the cluster version from 3.0 to 3.3
10.238451 I | etcdserver/api: enabled capabilities for version 3.3


^C21.192174 N | pkg/osutil: received interrupt signal, shutting down...
21.192459 I | etcdserver: 7339c4e5e833c029 starts leadership transfer from 7339c4e5e833c029 to 729934363faa4a24
21.192569 I | raft: 7339c4e5e833c029 [term 8] starts to transfer leadership to 729934363faa4a24
21.192619 I | raft: 7339c4e5e833c029 sends MsgTimeoutNow to 729934363faa4a24 immediately as 729934363faa4a24 already has up-to-date log
WARNING: 2018/05/17 12:45:21 grpc: addrConn.resetTransport failed to create client transport: connection error: desc = "transport: Error while dialing dial tcp: operation was canceled"; Reconnecting to {localhost:2379 0  <nil>}
WARNING: 2018/05/17 12:45:21 grpc: addrConn.transportMonitor exits due to: grpc: the connection is closing
21.193589 I | raft: 7339c4e5e833c029 [term: 8] received a MsgVote message with higher term from 729934363faa4a24 [term: 9]
21.193626 I | raft: 7339c4e5e833c029 became follower at term 9
21.193651 I | raft: 7339c4e5e833c029 [logterm: 8, index: 9, vote: 0] cast MsgVote for 729934363faa4a24 [logterm: 8, index: 9] at term 9
21.193675 I | raft: raft.node: 7339c4e5e833c029 lost leader 7339c4e5e833c029 at term 9
21.194424 I | raft: raft.node: 7339c4e5e833c029 elected leader 729934363faa4a24 at term 9
21.292898 I | etcdserver: 7339c4e5e833c029 finished leadership transfer from 7339c4e5e833c029 to 729934363faa4a24 (took 100.436391ms)
21.292975 I | rafthttp: stopping peer 729934363faa4a24...
21.293206 I | rafthttp: closed the TCP streaming connection with peer 729934363faa4a24 (stream MsgApp v2 writer)
21.293225 I | rafthttp: stopped streaming with peer 729934363faa4a24 (writer)
21.293437 I | rafthttp: closed the TCP streaming connection with peer 729934363faa4a24 (stream Message writer)
21.293459 I | rafthttp: stopped streaming with peer 729934363faa4a24 (writer)
21.293514 I | rafthttp: stopped HTTP pipelining with peer 729934363faa4a24
21.293590 W | rafthttp: lost the TCP streaming connection with peer 729934363faa4a24 (stream MsgApp v2 reader)
21.293610 I | rafthttp: stopped streaming with peer 729934363faa4a24 (stream MsgApp v2 reader)
21.293680 W | rafthttp: lost the TCP streaming connection with peer 729934363faa4a24 (stream Message reader)
21.293700 I | rafthttp: stopped streaming with peer 729934363faa4a24 (stream Message reader)
21.293711 I | rafthttp: stopped peer 729934363faa4a24
21.293720 I | rafthttp: stopping peer b548c2511513015...
21.293987 I | rafthttp: closed the TCP streaming connection with peer b548c2511513015 (stream MsgApp v2 writer)
21.294063 I | rafthttp: stopped streaming with peer b548c2511513015 (writer)
21.294467 I | rafthttp: closed the TCP streaming connection with peer b548c2511513015 (stream Message writer)
21.294561 I | rafthttp: stopped streaming with peer b548c2511513015 (writer)
21.294742 I | rafthttp: stopped HTTP pipelining with peer b548c2511513015
21.294867 W | rafthttp: lost the TCP streaming connection with peer b548c2511513015 (stream MsgApp v2 reader)
21.294892 I | rafthttp: stopped streaming with peer b548c2511513015 (stream MsgApp v2 reader)
21.294990 W | rafthttp: lost the TCP streaming connection with peer b548c2511513015 (stream Message reader)
21.295004 E | rafthttp: failed to read b548c2511513015 on stream Message (context canceled)
21.295013 I | rafthttp: peer b548c2511513015 became inactive
21.295024 I | rafthttp: stopped streaming with peer b548c2511513015 (stream Message reader)
21.295035 I | rafthttp: stopped peer b548c2511513015
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
+ --initial-cluster-state new \
+ --logger zap \
+ --log-outputs stderr
```

Новий etcd v3.4 опублікує свою інформацію в кластері. На цьому етапі кластер все ще працює за протоколом v3.3, який є найнижчою загальною версією.

> `{"level":"info","ts":1526586617.1647713,"caller":"membership/cluster.go:485","msg":"set initial cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"7339c4e5e833c029","cluster-version":"3.0"}`

> `{"level":"info","ts":1526586617.1648536,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.0"}`

> `{"level":"info","ts":1526586617.1649303,"caller":"membership/cluster.go:473","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"7339c4e5e833c029","from":"3.0","from":"3.3"}`

> `{"level":"info","ts":1526586617.1649797,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.3"}`

> `{"level":"info","ts":1526586617.2107732,"caller":"etcdserver/server.go:1770","msg":"published local member to cluster through raft","local-member-id":"7339c4e5e833c029","local-member-attributes":"{Name:s1 ClientURLs:[http://localhost:2379]}","request-path":"/0/members/7339c4e5e833c029/attributes","cluster-id":"7dee9ba76d59ed53","publish-timeout":7}`

Переконайтеся, що кожен учасник, а потім весь кластер, стає справним з новим двійковим файлом etcd v3.4:

```bash
etcdctl endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:32379 is healthy: successfully committed proposal: took = 2.337471ms
localhost:22379 is healthy: successfully committed proposal: took = 1.130717ms
localhost:2379 is healthy: successfully committed proposal: took = 2.124843ms
COMMENT
```

Неоновлені учасники будуть реєструвати попередження, як наведені нижче, доки весь кластер не буде оновлено до v3.4.

Це очікувано і припиниться після того, як усі учасники кластера etcd будуть оновлені до v3.4:

```
:41.942121 W | etcdserver: member 7339c4e5e833c029 has a higher version 3.4.0
:45.945154 W | etcdserver: the local etcd version 3.3.5 is not up-to-date
```

#### Крок 5: повторіть *крок 3* і *крок 4* для решти учасників {#step-5-repeat-step-3-and-step-4-for-rest-of-the-members}

Коли всі учасники будуть оновлені, кластер повідомить про успішне оновлення до 3.4:

Учасник 1:

> `{"level":"info","ts":1526586949.0920913,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.4"}`
> `{"level":"info","ts":1526586949.0921566,"caller":"etcdserver/server.go:2272","msg":"cluster version is updated","cluster-version":"3.4"}`

Учасник 2:

> `{"level":"info","ts":1526586949.092117,"caller":"membership/cluster.go:473","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"729934363faa4a24","from":"3.3","from":"3.4"}`
> `{"level":"info","ts":1526586949.0923078,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.4"}`

Учасник 3:

> `{"level":"info","ts":1526586949.0921423,"caller":"membership/cluster.go:473","msg":"updated cluster version","cluster-id":"7dee9ba76d59ed53","local-member-id":"b548c2511513015","from":"3.3","from":"3.4"}`
> `{"level":"info","ts":1526586949.0922918,"caller":"api/capability.go:76","msg":"enabled capabilities for version","cluster-version":"3.4"}`

```bash
endpoint health --endpoints=localhost:2379,localhost:22379,localhost:32379
<<COMMENT
localhost:2379 is healthy: successfully committed proposal: took = 492.834µs
localhost:22379 is healthy: successfully committed proposal: took = 1.015025ms
localhost:32379 is healthy: successfully committed proposal: took = 1.853077ms
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

[etcd-contact]: https://groups.google.com/g/etcd-dev

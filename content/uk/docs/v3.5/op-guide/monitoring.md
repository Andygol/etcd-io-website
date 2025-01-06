---
title: Моніторинг etcd
weight: 4500
description: Моніторинг etcd для перевірки справності системи та налагодження кластера
---

Кожен сервер etcd надає локальну інформацію про моніторинг на своєму клієнтському порту через точки доступу http. Дані моніторингу корисні як для перевірки справності системи, так і для налагодження кластера.

## Debug endpoint {#debug-endpoint}

Якщо встановлено `--log-level=debug`, сервер etcd експортує інформацію для налагодження на своєму клієнтському порту за шляхом `/debug`. Будьте обережні при встановленні `--log-level=debug`, оскільки це призведе до зниження продуктивності та великої кількості логів.

Точка доступу `/debug/pprof` є стандартною точкою доступу для профілювання виконання Go. Це можна використовувати для профілювання використання CPU, heap, mutex та goroutine. Наприклад, тут `go tool pprof` отримує топ 10 функцій, де etcd витрачає свій час:

```sh
$ go tool pprof http://localhost:2379/debug/pprof/profile
Fetching profile from http://localhost:2379/debug/pprof/profile
Please wait... (30s)
Saved profile in /home/etcd/pprof/pprof.etcd.localhost:2379.samples.cpu.001.pb.gz
Entering interactive mode (type "help" for commands)
(pprof) top10
310ms of 480ms total (64.58%)
Showing top 10 nodes out of 157 (cum >= 10ms)
    flat  flat%   sum%        cum   cum%
   130ms 27.08% 27.08%      130ms 27.08%  runtime.futex
    70ms 14.58% 41.67%       70ms 14.58%  syscall.Syscall
    20ms  4.17% 45.83%       20ms  4.17%  github.com/coreos/etcd/vendor/golang.org/x/net/http2/hpack.huffmanDecode
    20ms  4.17% 50.00%       30ms  6.25%  runtime.pcvalue
    20ms  4.17% 54.17%       50ms 10.42%  runtime.schedule
    10ms  2.08% 56.25%       10ms  2.08%  github.com/coreos/etcd/vendor/github.com/coreos/etcd/etcdserver.(*EtcdServer).AuthInfoFromCtx
    10ms  2.08% 58.33%       10ms  2.08%  github.com/coreos/etcd/vendor/github.com/coreos/etcd/etcdserver.(*EtcdServer).Lead
    10ms  2.08% 60.42%       10ms  2.08%  github.com/coreos/etcd/vendor/github.com/coreos/etcd/pkg/wait.(*timeList).Trigger
    10ms  2.08% 62.50%       10ms  2.08%  github.com/coreos/etcd/vendor/github.com/prometheus/client_golang/prometheus.(*MetricVec).hashLabelValues
    10ms  2.08% 64.58%       10ms  2.08%  github.com/coreos/etcd/vendor/golang.org/x/net/http2.(*Framer).WriteHeaders
```

Endpoint `/debug/requests` надає gRPC трасування та статистику продуктивності через вебоглядач. Наприклад, ось запит `Range` для ключа `abc`:

```log
When	Elapsed (s)
2017/08/18 17:34:51.999317 	0.000244 	/etcdserverpb.KV/Range
17:34:51.999382 	 .    65 	... RPC: from 127.0.0.1:47204 deadline:4.999377747s
17:34:51.999395 	 .    13 	... recv: key:"abc"
17:34:51.999499 	 .   104 	... OK
17:34:51.999535 	 .    36 	... sent: header:<cluster_id:14841639068965178418 member_id:10276657743932975437 revision:15 raft_term:17 > kvs:<key:"abc" create_revision:6 mod_revision:14 version:9 value:"asda" > count:1
```

## Metrics endpoint {#metrics-endpoint}

Кожен сервер etcd експортує метрики за шляхом `/metrics` на своєму клієнтському порту та за бажанням на місця, вказані параметром `--listen-metrics-urls`.

Метрики можна отримати за допомогою `curl`:

```sh
$ curl -L http://localhost:2379/metrics | grep -v debugging # ігнорувати нестабільні метрики налагодження

# HELP etcd_disk_backend_commit_duration_seconds Розподіл затримок коммітів, викликаних бекендом.
# TYPE etcd_disk_backend_commit_duration_seconds histogram
etcd_disk_backend_commit_duration_seconds_bucket{le="0.002"} 72756
etcd_disk_backend_commit_duration_seconds_bucket{le="0.004"} 401587
etcd_disk_backend_commit_duration_seconds_bucket{le="0.008"} 405979
etcd_disk_backend_commit_duration_seconds_bucket{le="0.016"} 406464
...
```

## Health Check {#health-check}

З версії v3.3.0, крім відповіді на точці доступу `/metrics`, будь-які місця, вказані параметром `--listen-metrics-urls`, також відповідатимуть на точку доступу `/health`. Це може бути корисно, якщо стандартна точка доступу налаштована на взаємну (клієнтську) TLS автентифікацію, але балансувальник навантаження або служба моніторингу все ще потребують доступу до перевірки справності.

З версії v3.5.12 додано дві нові точки доступу `/livez` та `/readyz`.

* точка доступу `/livez` відображає, чи процес живий або потребує перезапуску.
* точка доступу `/readyz` відображає, чи процес готовий обслуговувати трафік.

Деталі дизайну точок доступу задокументовані в [KEP](https://github.com/kubernetes/enhancements/tree/master/keps/sig-etcd/4331-livez-readyz).

Кожна точка доступу включає кілька індивідуальних перевірок справності, і ви можете використовувати параметр `verbose`, щоб вивести деталі перевірок та їх статус, наприклад

```bash
curl -k http://localhost:2379/readyz?verbose
```

і ви побачите відповідь, схожу на

```text
[+]data_corruption ok
[+]serializable_read ok
[+]linearizable_read ok
ok
```

HTTP API також підтримує виключення певних перевірок, наприклад

```bash
curl -k http://localhost:2379/readyz?exclude=data_corruption
```

## Prometheus {#prometheus}

Запуск служби моніторингу [Prometheus][prometheus] є найпростішим способом отримання та запису метрик etcd.

Спочатку встановіть Prometheus:

```sh
PROMETHEUS_VERSION="2.0.0"
wget https://github.com/prometheus/prometheus/releases/download/v$PROMETHEUS_VERSION/prometheus-$PROMETHEUS_VERSION.linux-amd64.tar.gz -O /tmp/prometheus-$PROMETHEUS_VERSION.linux-amd64.tar.gz
tar -xvzf /tmp/prometheus-$PROMETHEUS_VERSION.linux-amd64.tar.gz --directory /tmp/ --strip-components=1
/tmp/prometheus -version
```

Налаштуйте скрепер Prometheus для цільових точок доступу кластера etcd:

```sh
cat > /tmp/test-etcd.yaml <<EOF
global:
  scrape_interval: 10s
scrape_configs:
  - job_name: test-etcd
    static_configs:
    - targets: ['10.240.0.32:2379','10.240.0.33:2379','10.240.0.34:2379']
EOF
cat /tmp/test-etcd.yaml
```

Налаштуйте обробник Prometheus:

```sh
nohup /tmp/prometheus \
    -config.file /tmp/test-etcd.yaml \
    -web.listen-address ":9090" \
    -storage.local.path "test-etcd.data" >> /tmp/test-etcd.log  2>&1 &
```

Тепер Prometheus буде отримувати метрики etcd кожні 10 секунд.

### Alerting {#alerting}

Існує набір [стандартних сповіщень](https://github.com/etcd-io/etcd/tree/master/contrib/mixin) для кластерів etcd v3 для Prometheus.

{{% alert title="Примітка" color="info" %}}
Зверніть увагу, що мітки `job` можуть потребувати налаштування відповідно до конкретних потреб. Правила були написані для застосування до одного кластера, тому рекомендується вибирати мітки, унікальні для кластера.
{{% /alert %}}

### Grafana {#grafana}

[Grafana][grafana] має вбудовану підтримку Prometheus; просто додайте джерело даних Prometheus:

```text
Name:   test-etcd
Type:   Prometheus
Url:    http://localhost:9090
Access: proxy
```

Потім імпортуйте стандартний [шаблон інфопанелі etcd][template] та налаштуйте. Наприклад, якщо імʼя джерела даних Prometheus - `my-etcd`, значення поля `datasource` у JSON також повинні бути `my-etcd`.

Приклад інфопанелі:

![](/docs/v3.5/op-guide/etcd-sample-grafana.png)

## Distributed tracing {#distributed-tracing}

У версії v3.5 etcd додано підтримку розподіленого трасування за допомогою [OpenTelemetry](https://github.com/open-telemetry).

{{% alert title="Note" color="info" %}}
Ця функція все ще експериментальна і може змінюватися в будь-який час.
{{% /alert %}}

Щоб увімкнути цю експериментальну функцію, передайте параметр `--experimental-enable-distributed-tracing=true` серверу etcd, разом з параметром `--experimental-distributed-tracing-sampling-rate=<number>`, щоб вибрати, скільки зразків збирати на мільйон відрізків (span), стандартно швидкість вибірки становить `0`.

Налаштуйте розподілене трасування, запустивши сервер etcd з наступними додатковими параметрами:

- `--experimental-distributed-tracing-address` - (Необовʼязково) - "localhost:4317" - Адреса колектора трасування.

- `--experimental-distributed-tracing-service-name` - (Необовʼязково) - "etcd" - Імʼя служби розподіленого трасування, має бути однаковим для всіх екземплярів etcd.

- `--experimental-distributed-tracing-instance-id` - (Необовʼязково) - Ідентифікатор екземпляра, хоча необовʼязково, наполегливо рекомендується встановити, має бути унікальним для кожного екземпляра etcd.

Перед увімкненням розподіленого трасування переконайтеся, що у вас є точка доступу OpenTelemetry, якщо ця адреса відрізняється від стандартної, перевизначте її за допомогою параметра `--experimental-distributed-tracing-address`. Через те, що OpenTelemetry має різні способи запуску, зверніться до [документації колектора](https://opentelemetry.io/docs/collector/getting-started/), щоб дізнатися більше.

{{% alert title="Примітка" color="info" %}}
Існує ресурсне навантаження, як і з будь-яким сигналом спостережуваності, за нашими початковими вимірами це навантаження може становити від 2% до 4% навантаження на CPU.
{{% /alert %}}

[grafana]: http://grafana.org/
[prometheus]: https://prometheus.io/
[template]: ../grafana.json

---
title: Як перевірити стан кластера
linkTitle: Cтан кластера
description: Посібник з перевірки стану кластера etcd
weight: 1000
---

## Передумови {#prerequisites}

* Встановить [`etcd` та `etcdctl`](/docs/v3.8/install/)

## Перевірка загального стану {#check-overall-status}

`endpoint status` для перевірки загального стану кожної точки доступу, зазначеної у прапорці `--endpoints`:

```bash
etcdctl endpoint status (--endpoints=$ENDPOINTS|--cluster)
```

### Параметри {#options}

```bash
--cluster[=false]: використовувати всі точки доступу зі списку учасників кластера
```

## Перевірка справності {#check-health}

Використовуйте `endpoint health` для перевірки справності кожної точки доступу, зазначеної у прапорці `--endpoints`:

```bash
etcdctl endpoint health (--endpoints=$ENDPOINTS|--cluster)
```

### Параметри {#options-1}

```bash
--cluster[=false]: використовувати всі точки доступу зі списку учасників кластера
```

## Перевірка хешу KV {#check-kv-hash}

`endpoint hashkv` для перевірки хешу історії KV кожної точки доступу, зазначеної у прапорці `--endpoints`:

```bash
etcdctl endpoint hashkv (--endpoints=$ENDPOINTS|--cluster) [rev=$REV]
```

### Параметри {#options-2}

```bash
--cluster[=false]: використовувати всі точки доступу зі списку учасників кластера
--rev=0: максимальна ревізія для хешування (типово: остання ревізія)
```

## Параметри, успадковані від батьківських команд {#options-inherited-from-parent-commands}

```bash
--endpoints="127.0.0.1:2379": gRPC endpoints
-w, --write-out="simple": встановити формат виводу (fields, json, protobuf, simple, table)
```

### Приклади {#examples}

```shell
etcdctl --write-out=table --endpoints=$ENDPOINTS endpoint status

+------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 10.240.0.17:2379 | 4917a7ab173fabe7 |  3.5.0  |   45 kB |      true |      false |         4 |      16726 |              16726 |        |
| 10.240.0.18:2379 | 59796ba9cd1bcd72 |  3.5.0  |   45 kB |     false |      false |         4 |      16726 |              16726 |        |
| 10.240.0.19:2379 | 94df724b66343e6c |  3.5.0  |   45 kB |     false |      false |         4 |      16726 |              16726 |        |
+------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------|
```

```shell
etcdctl --endpoints=$ENDPOINTS endpoint health

10.240.0.17:2379 is healthy: successfully committed proposal: took = 3.345431ms
10.240.0.19:2379 is healthy: successfully committed proposal: took = 3.767967ms
10.240.0.18:2379 is healthy: successfully committed proposal: took = 4.025451ms
```

```shell
etcdctl --cluster endpoint hashkv  --write-out=table

+------------------+------------+---------------+
|     ENDPOINT     |    HASH    | HASH REVISION |
+------------------+------------+---------------+
| 10.240.0.17:2379 | 3892279174 |             3 |
| 10.240.0.18:2379 | 3892279174 |             3 |
| 10.240.0.19:2379 | 3892279174 |             3 |
+------------------+------------+---------------+
```

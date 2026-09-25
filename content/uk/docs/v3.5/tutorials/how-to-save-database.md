---
title: Як зберегти базу даних
linkTitle: Збереження БД
description: Посібник зі створення знімка бази даних etcd
weight: 1100
---

## Передумови {#prerequisites}

* [Встановити etcdctl, etcdutl](/docs/v3.5/install/)
* [Налаштувати локальний кластер](/docs/v3.5/dev-guide/local_cluster/)

## Знімок бази даних {#snapshot-a-database}

Використовуйте `snapshot` для збереження знімка бази даних etcd на певний момент часу:

```bash
etcdctl --endpoints=$ENDPOINT snapshot save DB_NAME
```

### Глобальні параметри {#global-options}

#### etcdctl

```bash
--endpoints=[127.0.0.1:2379], gRPC endpoints
```

Знімок можна запросити лише з одного вузла etcd, тому прапорець `--endpoints` повинен містити лише одну точку доступу.

#### etcdutl

```bash
-w, --write-out string   set the output format (fields, json, protobuf, simple, table) (default "simple")
```

### Приклад {#example}

![11_etcdctl_snapshot_2016051001](https://storage.googleapis.com/etcd/demo/11_etcdctl_snapshot_2016051001.gif)

```shell
ENDPOINTS=$HOST_1:2379
etcdctl --endpoints=$ENDPOINTS snapshot save my.db

Snapshot saved at my.db
```

```shell
etcdutl --write-out=table snapshot status my.db

+---------+----------+------------+------------+
|  HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+---------+----------+------------+------------+
| c55e8b8 |        9 |         13 | 25 kB      |
+---------+----------+------------+------------+
```

---
title: Читання з etcd
description: Читання значення в кластері etcd
weight: 200
---

## Передумови {#prerequisites}

- Встановіть `etcdctl`

## Процедура {#procedure}

Використовуйте субкоманду `get` для читання з etcd:

```shell
$ etcdctl --endpoints=$ENDPOINTS get foo
foo
Hello World!
$
```

де:
- `foo` — це потрібний ключ
- `Hello World!` — це отримане значення

Або, для форматованого виводу:

```log
$ etcdctl --endpoints=$ENDPOINTS --write-out="json" get foo
{"header":{"cluster_id":289318470931837780,"member_id":14947050114012957595,"revision":3,"raft_term":4,
"kvs":[{"key":"Zm9v","create_revision":2,"mod_revision":3,"version":2,"value":"SGVsbG8gV29ybGQh"}]}}
$
```

де `write-out="json"` призводить до виводу значення у форматі JSON (зверніть увагу, що ключ не повертається).

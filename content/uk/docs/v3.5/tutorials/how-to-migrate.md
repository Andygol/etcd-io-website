---
title: Як мігрувати etcd з v2 до v3
linkTitle: Міграція з v2 до v3
description: Посібник з міграції etcd з v2 до v3
weight: 1200
---

Використовуйте `migrate` для перетворення даних etcd з v2 до v3:

![12_etcdctl_migrate_2016061602](https://storage.googleapis.com/etcd/demo/12_etcdctl_migrate_2016061602.gif)


```shell
# записати ключ у сховище версії etcd 2
export ETCDCTL_API=2
etcdctl --endpoints=http://$ENDPOINT set foo bar

# прочитати ключ у etcd v2
etcdctl --endpoints=$ENDPOINTS --output="json" get foo

# зупинити вузол etcd для міграції, один за одним

# мігрувати дані v2
export ETCDCTL_API=3
etcdctl --endpoints=$ENDPOINT migrate --data-dir="default.etcd" --wal-dir="default.etcd/member/wal"

# перезапустити вузол etcd після міграції, один за одним

# підтвердити, що ключ було перенесено
etcdctl --endpoints=$ENDPOINTS get /foo
```

---
title: Як створити оренду
linkTitle: Створення оренди
description: Посібник зі створення оренди в etcd
weight: 700
---

Використовуйте `lease` для запису з часу існування (TTL, time-to-live) в etcd:

![07_etcdctl_lease_2016050501](https://storage.googleapis.com/etcd/demo/07_etcdctl_lease_2016050501.gif)

```shell
etcdctl --endpoints=$ENDPOINTS lease grant 300
# lease 2be7547fbc6a5afa granted with TTL(300s)

etcdctl --endpoints=$ENDPOINTS put sample value --lease=2be7547fbc6a5afa
etcdctl --endpoints=$ENDPOINTS get sample

etcdctl --endpoints=$ENDPOINTS lease keep-alive 2be7547fbc6a5afa
etcdctl --endpoints=$ENDPOINTS lease revoke 2be7547fbc6a5afa
# or after 300 seconds
etcdctl --endpoints=$ENDPOINTS get sample
```

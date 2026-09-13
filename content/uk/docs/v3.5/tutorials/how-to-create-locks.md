---
title: Як створювати блокування
linkTitle: Створення блокувань
description: Посібник зі створення розподілених блокувань в etcd
weight: 800
---

Використовуйте `lock` для розподіленого блокування:

![08_etcdctl_lock_2016050501](https://storage.googleapis.com/etcd/demo/08_etcdctl_lock_2016050501.gif)

```shell
etcdctl --endpoints=$ENDPOINTS lock mutex1

# another client with the same name blocks
etcdctl --endpoints=$ENDPOINTS lock mutex1
```

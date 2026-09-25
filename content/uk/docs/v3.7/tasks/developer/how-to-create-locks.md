---
title: Як створювати блокування
linkTitle: Створення блокувань
description: Посібник зі створення розподілених блокувань в etcd
weight: 800
---

LOCK здобуває розподілений мʼютекс із заданою назвою. Після отримання блокування воно утримується, доки etcdctl не буде завершено.

## Передумови {#prerequisites}

* Встановіть [`etcd` та `etcdctl`](/docs/v3.7/install/)

## Створення блокування {#creating-a-lock}

Використовуйте `lock` для розподіленого блокування:

![08_etcdctl_lock_2016050501](https://storage.googleapis.com/etcd/demo/08_etcdctl_lock_2016050501.gif)

```shell
etcdctl --endpoints=$ENDPOINTS lock mutex1
```

### Параметри {#options}

* endpoints — визначає розділений комами список адрес машин у кластері.
* ttl — час очікування в секундах для сесії блокування.

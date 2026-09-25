---
title: Як отримати ключі за префіксом
linkTitle: Отримання ключів
description: Посібник з вилучення ключів etcd за їх префіксом
weight: 300
---

## Передумови {#pre-requisites}

* [Встановити etcdctl](/docs/v3.5/install/)
* [Налаштувати локальний кластер](/docs/v3.5/dev-guide/local_cluster/)

## Отримати ключі за префіксом {#get-keys-by-prefix}

```bash
$ etcdctl --endpoints=$ENDPOINTS get PREFIX --prefix
```

### Глобальні параметри {#global-options}

```bash
--endpoints=[127.0.0.1:2379], gRPC endpoints
```

### Параметри {#options}

```bash
--prefix, отримати діапазон ключів з відповідним префіксом
```

### Приклад {#example}

![03_etcdctl_get_by_prefix_2016050501](https://storage.googleapis.com/etcd/demo/03_etcdctl_get_by_prefix_2016050501.gif)

```shell
etcdctl --endpoints=$ENDPOINTS put web1 value1
etcdctl --endpoints=$ENDPOINTS put web2 value2
etcdctl --endpoints=$ENDPOINTS put web3 value3

etcdctl --endpoints=$ENDPOINTS get web --prefix
```

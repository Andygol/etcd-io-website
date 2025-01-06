---
title: Як спостерігати за ключами
linkTitle: Спостереження за ключами
description: Посібник зі спостереження за ключами etcd
weight: 600
---

## Необхідні умови {#prerequisites}

* Встановіть [`etcd` та `etcdctl`](/docs/v3.5/install/)

## Спостереження за ключами {#watching-keys}

Використовуйте `watch`, щоб отримувати повідомлення про майбутні зміни:

```bash
etcdctl watch $KEY [$END_KEY]
```

### Опції {#options}

```bash
-i, --interactive[=false]: інтерактивний режим
--prefix[=false]: спостерігати за префіксом, якщо встановлено префікс
--rev=0: Ревізія для початку спостереження
--prev-kv[=false]: отримати попередню пару ключ-значення перед подією
--progress-notify[=false]: отримувати періодичні повідомлення про прогрес спостереження від сервера
```

### Опції, успадковані від батьківських команд {#options-inherited-from-parent-commands}

```bash
--endpoints="127.0.0.1:2379": gRPC endpoints
```

### Приклади {#examples}

![06_etcdctl_watch_2016050501](https://storage.googleapis.com/etcd/demo/06_etcdctl_watch_2016050501.gif)

```shell
etcdctl --endpoints=$ENDPOINTS watch stock1
etcdctl --endpoints=$ENDPOINTS put stock1 1000

etcdctl --endpoints=$ENDPOINTS watch stock --prefix
etcdctl --endpoints=$ENDPOINTS put stock1 10
etcdctl --endpoints=$ENDPOINTS put stock2 20
```

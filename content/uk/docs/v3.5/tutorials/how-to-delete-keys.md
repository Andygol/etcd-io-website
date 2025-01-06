---
title: Як видалити ключі
description: Описує спосіб видалення ключів etcd
weight: 400
---

## Передумови {#prerequisites}

* Встановіть [`etcd` та `etcdctl`](/docs/v3.5/install/)

## Додавання або видалення ключів {#add-or-delete-keys}

`del` для видалення вказаного ключа або діапазону ключів:

```bash
etcdctl del $KEY [$END_KEY]
```

### Опції {#options}

```bash
--prefix[=false]: видалити ключі з відповідним префіксом
--prev-kv[=false]: вивести видалені пари ключ-значення
--from-key[=false]: видалити ключі, які більші або рівні даному ключу, використовуючи байтове порівняння
--range[=false]: видалити діапазон ключів без затримки
```

### Опції, успадковані від батьківських команд {#options-inherited-from-parent-commands}

```bash
--endpoints="127.0.0.1:2379": gRPC endpoints
```

### Приклади {#examples}

![04_etcdctl_delete_2016050601](https://storage.googleapis.com/etcd/demo/04_etcdctl_delete_2016050601.gif)

```shell
etcdctl --endpoints=$ENDPOINTS put key myvalue
etcdctl --endpoints=$ENDPOINTS del key

etcdctl --endpoints=$ENDPOINTS put k1 value1
etcdctl --endpoints=$ENDPOINTS put k2 value2
etcdctl --endpoints=$ENDPOINTS del k --prefix
```

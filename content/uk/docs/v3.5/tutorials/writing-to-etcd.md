---
title: Запис до etcd
description: Додавання пари KV до кластера etcd
weight: 200
---

## Передумови {#prerequisites}

- Встановіть `etcdctl`

## Процедура {#procedure}

Використовуйте субкоманду `put` для запису пари ключ-значення:

```shell
etcdctl --endpoints=$ENDPOINTS put foo "Привіт Світ!"
```

де:

- `foo` є назвою ключа
- `"Hello World!"` є значенням у лапках

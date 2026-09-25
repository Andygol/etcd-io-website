---
title: Як проводити вибори лідера в кластері etcd
linkTitle: Вибори лідера
description: Кроки для проведення виборів лідера через клієнт etcdctl
weight: 900
---

## Передумови {#prerequisites}

- Переконайтеся, що [`etcd`](/docs/v3.8/install/) та [`etcdctl`](/docs/v3.8/install/) встановлені.
- Перевірте наявність активного кластера etcd.

## Проведення виборів лідера {#conduct-leader-election}

Команду `etcdctl` використовують для проведення виборів лідера в кластері etcd. Вона гарантує, що лише один клієнт може стати лідером за раз.

`etcdctl --endpoints=$ENDPOINTS elect <election-name> [proposal]`

```shell
etcdctl --endpoints=$ENDPOINTS elect election-name p1
```

### Параметри {#options}

- `--endpoints : $ENDPOINTS`

  Адреса кожного члена кластера etcd.

- `election-name` string

  Рядок ідентифікатора для виборів. Усі учасники, які конкурують за лідерство, мають використовувати однакову назву виборів.

- `leader-name` string

  Значення пропозиції нового лідера.

### Приклад {#example}

```shell
./etcdctl elect my-election proposal1
my-election/694d99fafea88404
proposal1

another election:
./etcdctl elect new-election proposal1
new-election/694d99fafea8840f
proposal1
```

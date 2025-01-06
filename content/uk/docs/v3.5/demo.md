---
title: Демонстрація
weight: 1100
description: Порядок роботи з кластером etcd
---

Ця серія прикладів показує основні процедури роботи з кластером etcd.


## Auth

`auth`,`user`,`role` для автентифікації:

```shell
export ETCDCTL_API=3
ENDPOINTS=localhost:2379

etcdctl --endpoints=${ENDPOINTS} role add root
etcdctl --endpoints=${ENDPOINTS} role get root

etcdctl --endpoints=${ENDPOINTS} user add root
etcdctl --endpoints=${ENDPOINTS} user grant-role root root
etcdctl --endpoints=${ENDPOINTS} user get root

etcdctl --endpoints=${ENDPOINTS} role add role0
etcdctl --endpoints=${ENDPOINTS} role grant-permission role0 readwrite foo
etcdctl --endpoints=${ENDPOINTS} user add user0
etcdctl --endpoints=${ENDPOINTS} user grant-role user0 role0

etcdctl --endpoints=${ENDPOINTS} auth enable
# тепер всі запити клієнтів проходять через автентифікацію

etcdctl --endpoints=${ENDPOINTS} --user=user0:123 put foo bar
etcdctl --endpoints=${ENDPOINTS} get foo
# у дозволі відмовлено, імʼя користувача порожнє, оскільки запит не видає запит на автентифікацію
etcdctl --endpoints=${ENDPOINTS} --user=user0:123 get foo
# user0 може прочитати ключ foo
etcdctl --endpoints=${ENDPOINTS} --user=user0:123 get foo1
```

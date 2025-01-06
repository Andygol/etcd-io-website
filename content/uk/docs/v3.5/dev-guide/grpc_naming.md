---
title: Іменування та виявлення gRPC
weight: 3500
description: "go-grpc: для вирішення gRPC кінцевих точок з бекендом etcd"
---

etcd надає gRPC резолвер для підтримки альтернативної системи імен, яка отримує точки доступу з etcd для виявлення gRPC сервісів. Основний механізм базується на спостереженні за оновленнями ключів з префіксом імені сервісу.

## Використання etcd discovery з go-grpc {#using-etcd-discovery-with-go-grpc}

Клієнт etcd надає gRPC резолвер для вирішення gRPC точок доступу з бекендом etcd. Резолвер ініціалізується клієнтом etcd:

```go
import (
	clientv3 "go.etcd.io/etcd/client/v3"
	etcdnaming "go.etcd.io/etcd/client/v3/naming/resolver"

	"google.golang.org/grpc"
)

...

cli, err := clientv3.NewFromURL("http://localhost:2379")
if err != nil {
    // ...
}
r, err := etcdnaming.NewBuilder(cli)
if err != nil {
    // ...
}
conn, gerr := grpc.Dial("my-service", grpc.WithResolvers(r), grpc.WithBlock(), ...)
```

## Управління точками доступу сервісу {#managing-service-endpoints}

Резолвер etcd обробляє всі ключі з префіксом цільового розвʼязання після "/" (наприклад, "foo/bar/my-service/") з JSON-кодованими (історично go-grpc `naming.Update`) значеннями як потенційні точки доступу сервісу. Точки доступу додаються до сервісу шляхом створення нових ключів і видаляються з сервісу шляхом видалення ключів.

### Додавання точки доступу {#adding-an-endpoint}

Нові точки доступу можуть бути додані до сервісу через `etcdctl`:

```sh
ETCDCTL_API=3 etcdctl put foo/bar/my-service/1.2.3.4 '{"Addr":"1.2.3.4","Metadata":"..."}'
```

Метод клієнта etcd `endpoints.Manager` також може реєструвати нові точки доступу з ключем, що відповідає `Addr`:

```go
em := endpoints.NewManager(client, "foo/bar/my-service")
err := em.AddEndpoint(context.TODO(),"foo/bar/my-service/e1", endpoints.Endpoint{Addr:"1.2.3.4"});
```

Щоб увімкнути балансування навантаження round-robin при підключенні до сервісу з кількома точками доступу, ви можете налаштувати підключення з внутрішнім балансувальником навантаження grpc round-robin:

```go
conn, gerr := grpc.Dial("etcd:///foo", grpc.WithResolvers(etcdResolver),
	grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`))
```

### Видалення точки доступу {#deleting-an-endpoint}

Хости можуть бути видалені з сервісу через `etcdctl`:

```sh
ETCDCTL_API=3 etcdctl del foo/bar/my-service/1.2.3.4
```

Метод клієнта etcd `endpoints.Manager` також підтримує видалення точок доступу:

```go
em := endpoints.NewManager(client, "foo/bar/my-service")
err := em.DeleteEndpoint(context.TODO(), "foo/bar/my-service/e1");
```

### Реєстрація точки доступу з орендою {#registering-an-endpoint-with-a-lease}

Реєстрація точки доступу з орендою гарантує, що якщо хост не зможе підтримувати keepalive heartbeat (наприклад, його машина вийде з ладу), він буде видалений з сервісу:

```sh
lease=`ETCDCTL_API=3 etcdctl lease grant 5 | cut -f2 -д' '`
ETCDCTL_API=3 etcdctl put --lease=$lease my-service/1.2.3.4 '{"Addr":"1.2.3.4","Metadata":"..."}'
ETCDCTL_API=3 etcdctl lease keep-alive $lease
```

У golang:

```go
lease, _ := client.Grant(context.TODO(), ttl)
em := endpoints.NewManager(client, "foo/bar/my-service")
err := em.AddEndpoint(context.TODO(), "foo/bar/my-service/e1", endpoints.Endpoint{Addr:"1.2.3.4"}, clientv3.WithLease(lease.ID));
```

### Атомарне оновлення точок доступу {#atomically-updating-endpoints}

Якщо потрібно змінити кілька точок доступу в одній транзакції, можна використовувати `endpoints.Manager` безпосередньо:

```go
em := endpoints.NewManager(c, "foo")

err := em.Update(context.TODO(), []*endpoints.UpdateWithOpts{
    endpoints.NewDeleteUpdateOpts("foo/bar/my-service/e1", endpoints.Endpoint{Addr: "1.2.3.4"}),
	endpoints.NewAddUpdateOpts("foo/bar/my-service/e1", endpoints.Endpoint{Addr: "1.2.3.14"})})
```

---
title: Запуск кластерів etcd у контейнерах
weight: 4200
description: Запуск etcd з Docker за допомогою статичного початкового завантаження
---
Наступний посібник показує, як запустити etcd з Docker за допомогою [статичного процесу початкового завантаження](../clustering/#static).

## Docker {#docker}

Щоб відкрити API etcd для клієнтів за межами хосту Docker, використовуйте IP-адресу хосту контейнера. Будь ласка, дивіться [`docker inspect`](https://docs.docker.com/engine/reference/commandline/inspect) для отримання додаткової інформації про те, як отримати IP-адресу. Альтернативно, вкажіть прапорець `--net=host` для команди `docker run`, щоб пропустити розміщення контейнера всередині окремого мережевого стека.

### Запуск одного вузла etcd {#running-a-single-node-etcd}

Використовуйте IP-адресу хосту при налаштуванні etcd:

```sh
export NODE1=192.168.1.21
```

Налаштуйте том Docker для зберігання даних etcd:

```sh
docker volume create --name etcd-data
export DATA_DIR="etcd-data"
```

Запустіть останню версію etcd:

```sh
REGISTRY=quay.io/coreos/etcd
# available from v3.2.5
REGISTRY=gcr.io/etcd-development/etcd

docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --volume=${DATA_DIR}:/etcd-data \
  --name etcd ${REGISTRY}:latest \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data --name node1 \
  --initial-advertise-peer-urls http://${NODE1}:2380 --listen-peer-urls http://0.0.0.0:2380 \
  --advertise-client-urls http://${NODE1}:2379 --listen-client-urls http://0.0.0.0:2379 \
  --initial-cluster node1=http://${NODE1}:2380
```

Перелічіть учасників кластера:

```sh
etcdctl --endpoints=http://${NODE1}:2379 member list
```

### Запуск кластера etcd з 3 вузлів {#running-a-3-node-etcd-cluster}

```sh
REGISTRY=quay.io/coreos/etcd
# available from v3.2.5
REGISTRY=gcr.io/etcd-development/etcd

# Для кожної машини
ETCD_VERSION=latest
TOKEN=my-etcd-token
CLUSTER_STATE=new
NAME_1=etcd-node-0
NAME_2=etcd-node-1
NAME_3=etcd-node-2
HOST_1=10.20.30.1
HOST_2=10.20.30.2
HOST_3=10.20.30.3
CLUSTER=${NAME_1}=http://${HOST_1}:2380,${NAME_2}=http://${HOST_2}:2380,${NAME_3}=http://${HOST_3}:2380
DATA_DIR=/var/lib/etcd

# Для вузла 1
THIS_NAME=${NAME_1}
THIS_IP=${HOST_1}
docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --volume=${DATA_DIR}:/etcd-data \
  --name etcd ${REGISTRY}:${ETCD_VERSION} \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data --name ${THIS_NAME} \
  --initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://0.0.0.0:2380 \
  --advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://0.0.0.0:2379 \
  --initial-cluster ${CLUSTER} \
  --initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}

# Для вузла 2
THIS_NAME=${NAME_2}
THIS_IP=${HOST_2}
docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --volume=${DATA_DIR}:/etcd-data \
  --name etcd ${REGISTRY}:${ETCD_VERSION} \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data --name ${THIS_NAME} \
  --initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://0.0.0.0:2380 \
  --advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://0.0.0.0:2379 \
  --initial-cluster ${CLUSTER} \
  --initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}

# Для вузла 3
THIS_NAME=${NAME_3}
THIS_IP=${HOST_3}
docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --volume=${DATA_DIR}:/etcd-data \
  --name etcd ${REGISTRY}:${ETCD_VERSION} \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data --name ${THIS_NAME} \
  --initial-advertise-peer-urls http://${THIS_IP}:2380 --listen-peer-urls http://0.0.0.0:2380 \
  --advertise-client-urls http://${THIS_IP}:2379 --listen-client-urls http://0.0.0.0:2379 \
  --initial-cluster ${CLUSTER} \
  --initial-cluster-state ${CLUSTER_STATE} --initial-cluster-token ${TOKEN}
```

Щоб запустити `etcdctl` з використанням версії API 3:

```sh
docker exec etcd /usr/local/bin/etcdctl put foo bar
```

## Bare Metal {#bare-metal}

Щоб розгорнути кластер etcd з 3 вузлів на фізичному обладнанні, приклади в [baremetal repo](https://github.com/coreos/coreos-baremetal/tree/master/examples) можуть бути корисними.

## Монтування тому сертифікатів {#mounting-a-certificate-volume}

Контейнер випуску etcd стандартно не включає сертифікати кореневих центрів сертифікації. Щоб використовувати HTTPS з сертифікатами, довіреними кореневим центром сертифікації (наприклад, для виявлення), змонтуйте теку сертифікатів у контейнер etcd:

```sh
REGISTRY=quay.io/coreos/etcd
# available from v3.2.5
REGISTRY=docker://gcr.io/etcd-development/etcd

rkt run \
  --insecure-options=image \
  --volume etcd-ssl-certs-bundle,kind=host,source=/etc/ssl/certs/ca-certificates.crt \
  --mount volume=etcd-ssl-certs-bundle,target=/etc/ssl/certs/ca-certificates.crt \
  ${REGISTRY}:latest -- --name my-name \
  --initial-advertise-peer-urls http://localhost:2380 --listen-peer-urls http://localhost:2380 \
  --advertise-client-urls http://localhost:2379 --listen-client-urls http://localhost:2379 \
  --discovery https://discovery.etcd.io/c11fbcdc16972e45253491a24fcf45e1
```

```sh
REGISTRY=quay.io/coreos/etcd
# available from v3.2.5
REGISTRY=gcr.io/etcd-development/etcd

docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --volume=/etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt \
  ${REGISTRY}:latest \
  /usr/local/bin/etcd --name my-name \
  --initial-advertise-peer-urls http://localhost:2380 --listen-peer-urls http://localhost:2380 \
  --advertise-client-urls http://localhost:2379 --listen-client-urls http://localhost:2379 \
  --discovery https://discovery.etcd.io/86a9ff6c8cb8b4c4544c1a2f88f8b801
```

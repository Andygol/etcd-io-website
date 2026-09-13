---
title: Встановлення
weight: 1150
description: Інструкції для встановлення etcd з попередньо зібраних двійкових файлів або з вихідного коду.
minGoVers: 1.21
---

## Вимоги {#requirements}

Перед встановленням etcd ознайомтеся з наступними сторінками:

- [Підтримувані платформи][Supported platforms]
- [Рекомендація щодо апаратного забезпечення][Hardware recommendations]

## Встановлення з попередньо зібраних двійкових файлів {#install-pre-built-binaries}

Найпростіший спосіб встановити etcd — з готових двійкових файлів:

1. Завантажте стиснутий файл архіву для вашої платформи з [Releases][], вибравши реліз [{{< param git_version_tag >}}][tagged-release] або пізніший.
2. Розпакуйте архівний файл. У результаті буде створено теку, яка міститиме двійкові файли.
3. Додайте виконувані двійкові файли до вашого шляху. Наприклад, перейменуйте та/або перемістіть двійкові файли до теки у вашому шляху (наприклад, `/usr/local/bin`), або додайте до шляху теку, створену на попередньому кроці.
4. За допомогою оболонки перевірте наявність `etcd` у вашій змінній path:

   ```sh
   $ etcd --version
   etcd Version: {{< psubstr git_version_tag 1 >}}
   ...
   ```

## Збирання з вихідного коду {#build-from-source}

Якщо у вас є [Go версії {{< param minGoVers >}}+][go], ви можете зібрати etcd з вихідного коду, виконавши ці кроки:

1. [Завантажте репозиторій etcd як zip-файл][download] і розпакуйте його, або клонуйте репозиторій за допомогою наступної команди.

   ```sh
   $ git clone -b {{< param git_version_tag >}} https://github.com/etcd-io/etcd.git
   ```

   Для збирання з `{{< param github_branch >}}@HEAD`, не використовуйте прапорець `-b  {{< param git_version_tag >}}`.

2. Change directory:

   ```sh
   $ cd etcd
   ```

3. Запустіть скрипт збірки:

   ```sh
   $ ./scripts/build.sh
   ```

   Двійкові файли знаходяться у теці `bin`.

4. Наприклад, додайте повний шлях до теки `bin` до вашої змінної path:

   ```sh
   $ export PATH="$PATH:`pwd`/bin"
   ```

5. Перевірте, чи є `etcd` на вашому шляху:

   ```sh
   $ etcd --version
   ```

## Встановлення за допомогою пакунків ОС {#installation-via-os-packages}

{{% alert title="Warning" color="warning" %}}

Встановлення etcd за допомогою менеджерів пакунків ОС може призвести до встановлення застарілих версій, оскільки вони не підтримуються автоматично і не підтримуються офіційно проєктом etcd. Тому використовуйте пакунки ОС з обережністю.*

{{% /alert %}}

Існують різні способи встановлення etcd на різні операційні системи, і це лише деякі приклади того, як це можна зробити.

### MacOS (Homebrew)

1. Оновіть homebrew:

```sh
$ brew update
```

1. Встановіть etcd:

```sh
$ brew install etcd
```

1. Перевірте встановлення

```sh
$ etcd --version
```

## Linux

{{% alert title="Warning" color="warning" %}}

Хоча встановлення etcd через офіційні репозиторії та менеджери пакунків багатьох основних дистрибутивів Linux можливе, опубліковані версії можуть бути значно застарілими. Отже, встановлювати у такий спосіб категорично не рекомендується.

{{% /alert %}}

Рекомендований спосіб встановлення etcd у Linux — за допомогою [попередньо зібраних бінарних файлів](#install-pre-built-binaries) або за допомогою Homebrew.

### Homebrew у Linux {#homebrew-on-linux}

[Homebrew може працювати на Linux][Homebrew can run on Linux], і може надавати останні версії програмного забезпечення.

- Передумови
  - Оновіть Homebrew:

    ```sh
    $ brew update
    ```

- Порядок дій
  - Встановлення за допомогою `brew`:

    ```sh
    $ brew install etcd
    ```

- Результат
  - Перевірте встановлення, отримавши версію:

    ```sh
    $ etcd --version
    etcd Version: {{< psubstr git_version_tag 1 >}}
    ...
    ```

## Docker {#docker}

etcd використовує [`gcr.io/etcd-development/etcd`](https://gcr.io/etcd-development/etcd) як основний реєстр контейнерів, а [`quay.io/coreos/etcd`](https://quay.io/coreos/etcd) як другорядний.

Для запуску etcd за допомогою Docker:

```sh
ETCD_VER={{< param git_version_tag >}}

rm -rf /tmp/etcd-data.tmp && mkdir -p /tmp/etcd-data.tmp && \
  docker rmi gcr.io/etcd-development/etcd:${ETCD_VER} || true && \
  docker run \
  -p 2379:2379 \
  -p 2380:2380 \
  --mount type=bind,source=/tmp/etcd-data.tmp,destination=/etcd-data \
  --name etcd-gcr-${ETCD_VER} \
  gcr.io/etcd-development/etcd:${ETCD_VER} \
  /usr/local/bin/etcd \
  --name s1 \
  --data-dir /etcd-data \
  --listen-client-urls http://0.0.0.0:2379 \
  --advertise-client-urls http://0.0.0.0:2379 \
  --listen-peer-urls http://0.0.0.0:2380 \
  --initial-advertise-peer-urls http://0.0.0.0:2380 \
  --initial-cluster s1=http://0.0.0.0:2380 \
  --initial-cluster-token tkn \
  --initial-cluster-state new \
  --log-level info \
  --logger zap \
  --log-outputs stderr

docker exec etcd-gcr-${ETCD_VER} /usr/local/bin/etcd --version
docker exec etcd-gcr-${ETCD_VER} /usr/local/bin/etcdctl version
docker exec etcd-gcr-${ETCD_VER} /usr/local/bin/etcdutl version
docker exec etcd-gcr-${ETCD_VER} /usr/local/bin/etcdctl endpoint health
docker exec etcd-gcr-${ETCD_VER} /usr/local/bin/etcdctl put foo bar
docker exec etcd-gcr-${ETCD_VER} /usr/local/bin/etcdctl get foo
```

## Встановлення як частина встановлення Kubernetes {#installation-as-part-of-kubernetes-installation}

- [Запуск etcd як Kubernetes StatefulSet][Running etcd as a Kubernetes StatefulSet]

## Перевірка встановлення {#installation-check}

Для дещо складнішої перевірки адекватності вашої інсталяції див. [Швидкий старт][Quickstart].

[download]: https://github.com/etcd-io/etcd/archive/{{< param git_version_tag >}}.zip
[go]: https://golang.org/doc/install
[Hardware recommendations]: {{< relref "op-guide/hardware" >}}
[Quickstart]: {{< relref "quickstart" >}}
[Running etcd as a Kubernetes StatefulSet]: {{< relref "op-guide/kubernetes" >}}
[releases]: https://github.com/etcd-io/etcd/releases/
[tagged-release]: https://github.com/etcd-io/etcd/releases/tag/{{< param git_version_tag >}}
[Supported platforms]: {{< relref "op-guide/supported-platform" >}}
[Homebrew can run on Linux]: <https://docs.brew.sh/Homebrew-on-Linux>
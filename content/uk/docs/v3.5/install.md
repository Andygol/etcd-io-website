---
title: Встановлення
weight: 1150
description: Інструкції для встановлення etcd з попередньо зібраних двійкових файлів або з вихідного коду.
minGoVers: 1.20
menu:
  main
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
   $ ./build.sh
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

## Встановлення за допомогою пакунків ОС {#install-via-os-packages}

*Застереження: встановлення etcd за допомогою менеджерів пакунків ОС може призвести до встановлення застарілих версій, оскільки вони не підтримуються автоматично і не підтримуються офіційно проєктом etcd. Тому використовуйте пакунки ОС з обережністю.*

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

Хоча встановлення etcd через офіційні репозиторії та менеджери пакунків багатьох основних дистрибутивів Linux можливе, опубліковані версії можуть бути значно застарілими. Отже, встановлювати у такий спосіб категорично не рекомендується.

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

## Встановлення як частина встановлення Kubernetes {#installation-as-part-of-kubernetes-installation}

- [Запуск etcd як Kubernetes StatefulSet][Running etcd as a Kubernetes StatefulSet]

## Встановлення на Kubernetes за допомогою statefulset або helm chart {#installation-on-kubernetes-using-a-statefulset-or-helm-chart}

Наразі проєкт etcd не підтримує чарти Helm, але ви можете скористатися інструкціями, наведеними e [Bitnami's etcd Helm chart].

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
[Bitnami's etcd Helm chart]: https://bitnami.com/stack/etcd/helm
[Homebrew can run on Linux]: <https://docs.brew.sh/Homebrew-on-Linux>

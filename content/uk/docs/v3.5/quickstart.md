---
title: Швидкий старт
weight: 900
description: Налаштуйте etcd менш ніж за 5 хвилин!
---

Дотримуйтесь цих інструкцій, щоб локально встановити, запустити і протестувати кластер, що складається з одного члена etcd:

 1. Встановіть etcd з попередньо зібраних двійкових файлів або з коду. Докладні відомості наведено у розділі [Встановлення][Install].

    {{% alert color="warning" %}}**Важливо**: Переконайтеся, що ви виконали останній крок інструкцій з встановлення, щоб переконатися, що `etcd` є у вашій змінній PATH.
    {{% /alert %}}

 2. Запустіть `etcd`:

    ```console
    $ etcd
    {"level":"info","ts":"2021-09-17T09:19:32.783-0400","caller":"etcdmain/etcd.go:72","msg":... }
    ⋮
    ```

    {{% alert color="info" %}}**Примітка**: Вивід, який створює `etcd` є [журналами](../op-guide/configuration/#logging) &mdash; логи інформаційного рівня можна ігнорувати. {{% /alert %}}

 3. В **іншому терміналі**, скористайтесь`etcdctl` для встановлення ключа:

    ```console
    $ etcdctl put greeting "Hello, etcd"
    OK
    ```

 4. З того ж терміналу отримайте ключ:

    ```console
    $ etcdctl get greeting
    greeting
    Hello, etcd
    ```

## Що далі {#whats-next}

Дізнайтеся більше про те, як налаштувати та використовувати etcd на наступних сторінках:

- Ознайомтесь з gRPC [API][].
- Розгорніть [кластер з кількох машин][clustering].
- Дізнайтесь як [налаштовувати][configure] etcd.
- Знайдіть [привʼязки та інструменти для мов][integrations].
- Використання TLS для забезпечення [захисту кластера etcd][security].
- [Налаштуйте etcd][tuning].

[api]: /uk/docs/{{< param version >}}/learning/api
[clustering]: /uk/docs/{{< param version >}}/op-guide/clustering
[configure]: /uk/docs/{{< param version >}}/op-guide/configuration
[integrations]: /uk/docs/{{< param version >}}/integrations
[security]: /uk/docs/{{< param version >}}/op-guide/security
[tuning]: /uk/docs/{{< param version >}}/tuning
[Install]: ../install/

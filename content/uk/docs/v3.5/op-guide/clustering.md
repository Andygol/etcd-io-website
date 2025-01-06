---
title: Посібник з кластеризації
weight: 4150
description: "Запуск кластера etcd: Статичний, etcd Discovery та DNS Discovery"
---

## Огляд {#overview}

Запуск кластера etcd статично вимагає, щоб кожен учасник знав іншого в кластері. У ряді випадків IP-адреси учасників кластера можуть бути невідомі заздалегідь. У таких випадках кластер etcd можна запустити за допомогою служби виявлення.

Після запуску кластера etcd додавання або видалення учасників здійснюється через [переконфігурацію під час виконання][runtime-conf]. Щоб краще зрозуміти дизайн переконфігурації під час виконання, рекомендуємо прочитати [документ про дизайн переконфігурації під час виконання][runtime-reconf-design].

Цей посібник охоплює наступні механізми запуску кластера etcd:

* [Статичний](#static)
* [etcd Discovery](#etcd-discovery)
* [DNS Discovery](#dns-discovery)

Кожен з механізмів запуску буде використаний для створення кластера etcd з трьох машин з наступними деталями:

|Назва|Адреса|Імʼя хосту|
|------|---------|------------------|
|infra0|10.0.1.10|infra0.example.com|
|infra1|10.0.1.11|infra1.example.com|
|infra2|10.0.1.12|infra2.example.com|

## Статичний {#static}

Оскільки ми знаємо учасників кластера, їхні адреси та розмір кластера перед запуском, ми можемо використовувати офлайн конфігурацію запуску, встановивши прапорець `initial-cluster`. Кожна машина отримає або наступні змінні середовища, або командний рядок:

```sh
ETCD_INITIAL_CLUSTER="infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380"
ETCD_INITIAL_CLUSTER_STATE=new
```

```sh
--initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
--initial-cluster-state new
```

Зверніть увагу, що URL-адреси, зазначені в `initial-cluster`, є _оголошеними URL-адресами партнерів_, тобто вони повинні відповідати значенню `initial-advertise-peer-urls` на відповідних вузлах.

Якщо запускати кілька кластерів (або створювати та знищувати один кластер) з однаковою конфігурацією для тестування, наполегливо рекомендується, щоб кожен кластер мав унікальний `initial-cluster-token`. Це дозволяє etcd генерувати унікальні ідентифікатори кластерів та ідентифікатори учасників для кластерів, навіть якщо вони мають однакову конфігурацію. Це може захистити etcd від взаємодії між кластерами, що може призвести до пошкодження кластерів.

etcd слухає на [`listen-client-urls`][conf-listen-client] для приймання клієнтського трафіку. Учасник etcd оголошує URL-адреси, зазначені в [`advertise-client-urls`][conf-adv-client], іншим учасникам, проксі та клієнтам. Переконайтеся, що `advertise-client-urls` доступні з передбачуваних клієнтів. Поширеною помилкою є встановлення `advertise-client-urls` на localhost або залишення його стандартним, якщо віддалені клієнти повинні досягати etcd.

На кожній машині запустіть etcd з цими прапорцями:

```sh
$ etcd --name infra0 --initial-advertise-peer-urls http://10.0.1.10:2380 \
  --listen-peer-urls http://10.0.1.10:2380 \
  --listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.10:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state new
```

```sh
$ etcd --name infra1 --initial-advertise-peer-urls http://10.0.1.11:2380 \
  --listen-peer-urls http://10.0.1.11:2380 \
  --listen-client-urls http://10.0.1.11:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.11:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state new
```

```sh
$ etcd --name infra2 --initial-advertise-peer-urls http://10.0.1.12:2380 \
  --listen-peer-urls http://10.0.1.12:2380 \
  --listen-client-urls http://10.0.1.12:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.12:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state new
```

Параметри командного рядка, що починаються з `--initial-cluster`, будуть ігноруватися при наступних запусках etcd. Не соромтеся видаляти змінні середовища або прапорці командного рядка після початкового процесу запуску. Якщо конфігурацію потрібно змінити пізніше (наприклад, додати або видалити учасників з кластера), дивіться посібник з [переконфігурації під час виконання][runtime-conf].

### TLS {#tls}

etcd підтримує зашифроване спілкування через протокол TLS. Канали TLS можуть використовуватися для зашифрованого внутрішнього кластерного спілкування між партнерами, а також для зашифрованого клієнтського трафіку. Цей розділ надає приклади налаштування кластера з TLS для партнерів та клієнтів. Додаткову інформацію про підтримку TLS в etcd можна знайти в [посібнику з безпеки][security-guide].

#### Самопідписні сертифікати {#self-signed-certificates}

Кластер, що використовує самопідписні сертифікати, як шифрує трафік, так і автентифікує свої зʼєднання. Щоб запустити кластер з самопідписними сертифікатами, кожен учасник кластера повинен мати унікальну пару ключів (`member.crt`, `member.key`), підписану спільним сертифікатом CA кластера (`ca.crt`) як для зʼєднань між партнерами, так і для клієнтських зʼєднань. Сертифікати можна згенерувати, дотримуючись прикладу налаштування TLS в etcd [TLS setup][tls-setup].

На кожній машині etcd буде запущено з цими прапорцями:

```sh
$ etcd --name infra0 --initial-advertise-peer-urls https://10.0.1.10:2380 \
  --listen-peer-urls https://10.0.1.10:2380 \
  --listen-client-urls https://10.0.1.10:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.0.1.10:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=https://10.0.1.10:2380,infra1=https://10.0.1.11:2380,infra2=https://10.0.1.12:2380 \
  --initial-cluster-state new \
  --client-cert-auth --trusted-ca-file=/path/to/ca-client.crt \
  --cert-file=/path/to/infra0-client.crt --key-file=/path/to/infra0-client.key \
  --peer-client-cert-auth --peer-trusted-ca-file=ca-peer.crt \
  --peer-cert-file=/path/to/infra0-peer.crt --peer-key-file=/path/to/infra0-peer.key
```

```sh
$ etcd --name infra1 --initial-advertise-peer-urls https://10.0.1.11:2380 \
  --listen-peer-urls https://10.0.1.11:2380 \
  --listen-client-urls https://10.0.1.11:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.0.1.11:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=https://10.0.1.10:2380,infra1=https://10.0.1.11:2380,infra2=https://10.0.1.12:2380 \
  --initial-cluster-state new \
  --client-cert-auth --trusted-ca-file=/path/to/ca-client.crt \
  --cert-file=/path/to/infra1-client.crt --key-file=/path/to/infra1-client.key \
  --peer-client-cert-auth --peer-trusted-ca-file=ca-peer.crt \
  --peer-cert-file=/path/to/infra1-peer.crt --peer-key-file=/path/to/infra1-peer.key
```

```sh
$ etcd --name infra2 --initial-advertise-peer-urls https://10.0.1.12:2380 \
  --listen-peer-urls https://10.0.1.12:2380 \
  --listen-client-urls https://10.0.1.12:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.0.1.12:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=https://10.0.1.10:2380,infra1=https://10.0.1.11:2380,infra2=https://10.0.1.12:2380 \
  --initial-cluster-state new \
  --client-cert-auth --trusted-ca-file=/path/to/ca-client.crt \
  --cert-file=/path/to/infra2-client.crt --key-file=/path/to/infra2-client.key \
  --peer-client-cert-auth --peer-trusted-ca-file=ca-peer.crt \
  --peer-cert-file=/path/to/infra2-peer.crt --peer-key-file=/path/to/infra2-peer.key
```

#### Автоматичні сертифікати {#automatic-certificates}

Якщо кластер потребує зашифрованого спілкування, але не вимагає автентифікованих зʼєднань, etcd можна налаштувати на автоматичне генерування ключів. Під час ініціалізації кожен учасник створює свій набір ключів на основі своїх оголошених IP-адрес та хостів.

На кожній машині etcd буде запущено з цими прапорцями:

```sh
$ etcd --name infra0 --initial-advertise-peer-urls https://10.0.1.10:2380 \
  --listen-peer-urls https://10.0.1.10:2380 \
  --listen-client-urls https://10.0.1.10:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.0.1.10:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=https://10.0.1.10:2380,infra1=https://10.0.1.11:2380,infra2=https://10.0.1.12:2380 \
  --initial-cluster-state new \
  --auto-tls \
  --peer-auto-tls
```

```sh
$ etcd --name infra1 --initial-advertise-peer-urls https://10.0.1.11:2380 \
  --listen-peer-urls https://10.0.1.11:2380 \
  --listen-client-urls https://10.0.1.11:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.0.1.11:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=https://10.0.1.10:2380,infra1=https://10.0.1.11:2380,infra2=https://10.0.1.12:2380 \
  --initial-cluster-state new \
  --auto-tls \
  --peer-auto-tls
```

```sh
$ etcd --name infra2 --initial-advertise-peer-urls https://10.0.1.12:2380 \
  --listen-peer-urls https://10.0.1.12:2380 \
  --listen-client-urls https://10.0.1.12:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://10.0.1.12:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster infra0=https://10.0.1.10:2380,infra1=https://10.0.1.11:2380,infra2=https://10.0.1.12:2380 \
  --initial-cluster-state new \
  --auto-tls \
  --peer-auto-tls
```

### Випадки помилок {#error-cases}

У наступному прикладі ми не включили наш новий хост до списку перерахованих вузлів. Якщо це новий кластер, вузол _повинен_ бути доданий до списку початкових учасників кластера.

```sh
$ etcd --name infra1 --initial-advertise-peer-urls http://10.0.1.11:2380 \
  --listen-peer-urls https://10.0.1.11:2380 \
  --listen-client-urls http://10.0.1.11:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.11:2379 \
  --initial-cluster infra0=http://10.0.1.10:2380 \
  --initial-cluster-state new
etcd: infra1 not listed in the initial cluster config
exit 1
```

У цьому прикладі ми намагаємося зіставити вузол (infra0) на іншій адресі (127.0.0.1:2380), ніж його адреса у списку кластера (10.0.1.10:2380). Якщо цей вузол повинен слухати на кількох адресах, усі адреси _повинні_ бути відображені в директиві конфігурації "initial-cluster".

```sh
$ etcd --name infra0 --initial-advertise-peer-urls http://127.0.0.1:2380 \
  --listen-peer-urls http://10.0.1.10:2380 \
  --listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.10:2379 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra2=http://10.0.1.12:2380 \
  --initial-cluster-state=new
etcd: error setting up initial cluster: infra0 has different advertised URLs in the cluster and advertised peer URLs list
exit 1
```

Якщо партнер налаштований з іншим набором аргументів конфігурації та намагається приєднатися до цього кластера, etcd повідомить про невідповідність ідентифікатора кластера та завершить роботу.

```sh
$ etcd --name infra3 --initial-advertise-peer-urls http://10.0.1.13:2380 \
  --listen-peer-urls http://10.0.1.13:2380 \
  --listen-client-urls http://10.0.1.13:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.13:2379 \
  --initial-cluster infra0=http://10.0.1.10:2380,infra1=http://10.0.1.11:2380,infra3=http://10.0.1.13:2380 \
  --initial-cluster-state=new
etcd: conflicting cluster ID to the target cluster (c6ab534d07e8fcc4 != bc25ea2a74fb18b0). Exiting.
exit 1
```

## Виявлення {#discovery}

У ряді випадків IP-адреси партнерів кластера можуть бути невідомі заздалегідь. Це поширено при використанні хмарних провайдерів або коли мережа використовує DHCP. У таких випадках, замість вказівки статичної конфігурації, використовуйте наявний кластер etcd для запуску нового. Цей процес називається "виявленням".

Існує два методи, які можна використовувати для виявлення:

* служба виявлення etcd
* записи DNS SRV

### etcd discovery {#etcd-discovery}

Щоб краще зрозуміти дизайн протоколу служби виявлення, рекомендуємо прочитати [документацію протоколу служби виявлення][discovery-proto].

#### Термін дії URL-адреси виявлення {#lifetime-of-a-discovery-url}

URL-адреса виявлення ідентифікує унікальний кластер etcd. Замість повторного використання наявної URL-адреси виявлення, кожен екземпляр etcd ділиться новою URL-адресою виявлення для запуску нового кластера.

Щобільше, URL-адреси виявлення повинні використовуватися ТІЛЬКИ для початкового запуску кластера. Щоб змінити склад кластера після його запуску, дивіться посібник з [переконфігурації під час виконання][runtime-conf].

#### Власна служба виявлення etcd {#custom-etcd-discovery-service}

Виявлення використовує наявний кластер для запуску. Якщо використовується приватний кластер etcd, створіть URL-адресу таким чином:

```sh
$ curl -X PUT https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83/_config/size -d value=3
```

Встановивши ключ розміру для URL-адреси, створюється URL-адреса виявлення з очікуваним розміром кластера 3.

URL-адреса для використання в цьому випадку буде `https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83`, і учасники etcd будуть використовувати теку `https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83` для реєстрації під час запуску.

**Кожен учасник повинен мати вказаний інший прапорець імені. `Hostname` або `machine-id` можуть бути хорошим вибором. Інакше виявлення не вдасться через дублювання імен.**

Тепер ми запускаємо etcd з відповідними прапорцями для кожного учасника:

```sh
$ etcd --name infra0 --initial-advertise-peer-urls http://10.0.1.10:2380 \
  --listen-peer-urls http://10.0.1.10:2380 \
  --listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.10:2379 \
  --discovery https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83
```

```sh
$ etcd --name infra1 --initial-advertise-peer-urls http://10.0.1.11:2380 \
  --listen-peer-urls http://10.0.1.11:2380 \
  --listen-client-urls http://10.0.1.11:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.11:2379 \
  --discovery https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83
```

```sh
$ etcd --name infra2 --initial-advertise-peer-urls http://10.0.1.12:2380 \
  --listen-peer-urls http://10.0.1.12:2380 \
  --listen-client-urls http://10.0.1.12:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.12:2379 \
  --discovery https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83
```

Це призведе до того, що кожен учасник зареєструється в користувацькій службі виявлення etcd і почне кластер після реєстрації всіх машин.

#### Публічна служба виявлення etcd {#public-etcd-discovery-service}

Якщо немає наявного кластера, використовуйте публічну службу виявлення, розміщену на `discovery.etcd.io`. Щоб створити приватну URL-адресу виявлення за допомогою точки доступу "new", використовуйте команду:

```sh
$ curl https://discovery.etcd.io/new?size=3
https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
```

Це створить кластер з початковим розміром 3 учасників. Якщо розмір не вказано, використовується стандартне значення 3.

```sh
ETCD_DISCOVERY=https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
```

```sh
--discovery https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
```

**Кожен учасник повинен мати вказаний інший прапорець імені, інакше виявлення не вдасться через дублювання імен. `Hostname` або `machine-id` можуть бути хорошим вибором.**

Тепер ми запускаємо etcd з відповідними прапорцями для кожного учасника:

```sh
$ etcd --name infra0 --initial-advertise-peer-urls http://10.0.1.10:2380 \
  --listen-peer-urls http://10.0.1.10:2380 \
  --listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.10:2379 \
  --discovery https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
```

```sh
$ etcd --name infra1 --initial-advertise-peer-urls http://10.0.1.11:2380 \
  --listen-peer-urls http://10.0.1.11:2380 \
  --listen-client-urls http://10.0.1.11:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.11:2379 \
  --discovery https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
```

```sh
$ etcd --name infra2 --initial-advertise-peer-urls http://10.0.1.12:2380 \
  --listen-peer-urls http://10.0.1.12:2380 \
  --listen-client-urls http://10.0.1.12:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.12:2379 \
  --discovery https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
```

Це призведе до того, що кожен учасник зареєструється в службі виявлення і почне кластер після реєстрації всіх учасників.

Використовуйте змінну середовища `ETCD_DISCOVERY_PROXY`, щоб змусити etcd використовувати HTTP-проксі для підключення до служби виявлення.

#### Випадки помилок та попереджень {#error-and-warning-cases}

##### Помилки сервера виявлення {#discovery-server-errors}

```sh
$ etcd --name infra0 --initial-advertise-peer-urls http://10.0.1.10:2380 \
  --listen-peer-urls http://10.0.1.10:2380 \
  --listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.10:2379 \
  --discovery https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
etcd: error: the cluster doesn’t have a size configuration value in https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de/_config
exit 1
```

##### Попередження {#warnings}

Це нешкідливе попередження, що вказує на те, що URL-адреса виявлення буде ігноруватися на цій машині.

```sh
$ etcd --name infra0 --initial-advertise-peer-urls http://10.0.1.10:2380 \
  --listen-peer-urls http://10.0.1.10:2380 \
  --listen-client-urls http://10.0.1.10:2379,http://127.0.0.1:2379 \
  --advertise-client-urls http://10.0.1.10:2379 \
  --discovery https://discovery.etcd.io/3e86b59982e49066c5d813af1c2e2579cbf573de
etcdserver: discovery token ignored since a cluster has already been initialized. Valid log found at /var/lib/etcd
```

### DNS discovery {#dns-discovery}

DNS [SRV записи][rfc-srv] можуть використовуватися як механізм виявлення. Прапорець `--discovery-srv` можна використовувати для встановлення доменного імені DNS, де можна знайти записи виявлення SRV. Встановлення `--discovery-srv example.com` призводить до пошуку записів DNS SRV у зазначеному порядку:

* _etcd-server-ssl._tcp.example.com
* _etcd-server._tcp.example.com

Якщо знайдено `_etcd-server-ssl._tcp.example.com`, etcd спробує процес запуску через TLS.

Щоб допомогти клієнтам виявити кластер etcd, наступні записи DNS SRV шукаються у зазначеному порядку:

* _etcd-client._tcp.example.com
* _etcd-client-ssl._tcp.example.com

Якщо знайдено `_etcd-client-ssl._tcp.example.com`, клієнти спробують спілкуватися з кластером etcd через SSL/TLS.

Якщо etcd використовує TLS, запис виявлення SRV (наприклад, `example.com`) повинен бути включений у поле DNS SAN сертифіката SSL разом з імʼям хосту, інакше кластеризація не вдасться з повідомленнями в журналі, такими як наступні:

```log
[...] rejected connection from "10.0.1.11:53162" (error "remote error: tls: bad certificate", ServerName "example.com")
```

Якщо etcd використовує TLS без власного центру сертифікації, домен виявлення (наприклад, example.com) повинен відповідати домену запису SRV (наприклад, infra1.example.com). Це для запобігання атакам, які підробляють записи SRV, щоб вказувати на інший домен; домен матиме дійсний сертифікат під PKI, але буде контролюватися невідомою третьою стороною.

Прапорець `-discovery-srv-name` додатково налаштовує суфікс до імені SRV, яке запитується під час виявлення. Використовуйте цей прапорець, щоб розрізняти кілька кластерів etcd під одним доменом. Наприклад, якщо встановлено `discovery-srv=example.com` та `-discovery-srv-name=foo`, виконуються наступні запити DNS SRV:

* _etcd-server-ssl-foo._tcp.example.com
* _etcd-server-foo._tcp.example.com

#### Створення записів DNS SRV {#create-dns-srv-records}

```sh
$ dig +noall +answer SRV _etcd-server._tcp.example.com
_etcd-server._tcp.example.com. 300 IN  SRV  0 0 2380 infra0.example.com.
_etcd-server._tcp.example.com. 300 IN  SRV  0 0 2380 infra1.example.com.
_etcd-server._tcp.example.com. 300 IN  SRV  0 0 2380 infra2.example.com.
```

```sh
$ dig +noall +answer SRV _etcd-client._tcp.example.com
_etcd-client._tcp.example.com. 300 IN SRV 0 0 2379 infra0.example.com.
_etcd-client._tcp.example.com. 300 IN SRV 0 0 2379 infra1.example.com.
_etcd-client._tcp.example.com. 300 IN SRV 0 0 2379 infra2.example.com.
```

```sh
$ dig +noall +answer infra0.example.com infra1.example.com infra2.example.com
infra0.example.com.  300  IN  A  10.0.1.10
infra1.example.com.  300  IN  A  10.0.1.11
infra2.example.com.  300  IN  A  10.0.1.12
```

#### Запуск кластера etcd за допомогою DNS {#bootstrap-the-etcd-cluster-using-dns}

Учасники кластера etcd можуть оголошувати доменні імена або IP-адреси, процес запуску буде вирішувати записи DNS A. З версії 3.2 (3.1 друкує попередження) `--listen-peer-urls` та `--listen-client-urls` відхилятимуть доменне імʼя для привʼязки мережевого інтерфейсу.

Вирішена адреса в `--initial-advertise-peer-urls` _повинна відповідати_ одній з вирішених адрес у цільових записах SRV. Учасник etcd читає вирішену адресу, щоб визначити, чи належить вона до кластера, визначеного в записах SRV.

```sh
$ etcd --name infra0 \
--discovery-srv example.com \
--initial-advertise-peer-urls http://infra0.example.com:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster-state new \
--advertise-client-urls http://infra0.example.com:2379 \
--listen-client-urls http://0.0.0.0:2379 \
--listen-peer-urls http://0.0.0.0:2380
```

```sh
$ etcd --name infra1 \
--discovery-srv example.com \
--initial-advertise-peer-urls http://infra1.example.com:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster-state new \
--advertise-client-urls http://infra1.example.com:2379 \
--listen-client-urls http://0.0.0.0:2379 \
--listen-peer-urls http://0.0.0.0:2380
```

```sh
$ etcd --name infra2 \
--discovery-srv example.com \
--initial-advertise-peer-urls http://infra2.example.com:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster-state new \
--advertise-client-urls http://infra2.example.com:2379 \
--listen-client-urls http://0.0.0.0:2379 \
--listen-peer-urls http://0.0.0.0:2380
```

Кластер також може запускатися за допомогою IP-адрес замість доменних імен:

```sh
$ etcd --name infra0 \
--discovery-srv example.com \
--initial-advertise-peer-urls http://10.0.1.10:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster-state new \
--advertise-client-urls http://10.0.1.10:2379 \
--listen-client-urls http://10.0.1.10:2379 \
--listen-peer-urls http://10.0.1.10:2380
```

```sh
$ etcd --name infra1 \
--discovery-srv example.com \
--initial-advertise-peer-urls http://10.0.1.11:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster-state new \
--advertise-client-urls http://10.0.1.11:2379 \
--listen-client-urls http://10.0.1.11:2379 \
--listen-peer-urls http://10.0.1.11:2380
```

```sh
$ etcd --name infra2 \
--discovery-srv example.com \
--initial-advertise-peer-urls http://10.0.1.12:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-cluster-state new \
--advertise-client-urls http://10.0.1.12:2379 \
--listen-client-urls http://10.0.1.12:2379 \
--listen-peer-urls http://10.0.1.12:2380
```

З версії v3.1.0 (крім v3.2.9), коли `etcd --discovery-srv=example.com` налаштовано з TLS, сервер буде автентифікувати партнерів/клієнтів тільки тоді, коли надані сертифікати мають кореневий домен `example.com` як запис у полі Subject Alternative Name (SAN). Дивіться [Примітки для DNS SRV][security-guide-dns-srv].

### Шлюз {#gateway}

Шлюз etcd — це простий TCP-проксі, який пересилає мережеві дані до кластера etcd. Будь ласка, прочитайте [посібник шлюзу][gateway] для отримання додаткової інформації.

### Проксі {#proxy}

Коли встановлено прапорець `--proxy`, etcd працює в [режимі проксі][proxy]. Цей режим проксі підтримує тільки API v2 etcd; немає планів підтримувати API v3. Натомість для підтримки API v3 буде новий проксі з розширеними функціями після випуску etcd 3.0.

Щоб налаштувати кластер etcd з проксі v2 API, будь ласка, прочитайте [документ про кластеризацію у випуску etcd 2.3][clustering_etcd2].

[clustering_etcd2]: https://github.com/etcd-io/etcd/blob/release-2.3/Documentation/clustering.md
[conf-adv-client]: ../configuration/#clustering
[conf-listen-client]: ../configuration/#member
[discovery-proto]: ../../dev-internal/discovery_protocol/
[gateway]: ../gateway/
[proxy]: https://github.com/etcd-io/etcd/blob/release-2.3/Documentation/proxy.md
[rfc-srv]: http://www.ietf.org/rfc/rfc2052.txt
[runtime-conf]: ../runtime-configuration/
[runtime-reconf-design]: ../runtime-reconf-design/
[security-guide-dns-srv]: ../security/#notes-for-dns-srv
[security-guide]: ../security/
[tls-setup]: https://github.com/etcd-io/etcd/tree/master/hack/tls-setup

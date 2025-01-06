---
title: Модель транспортної безпеки
weight: 4125
description: Захист даних під час передачі
---

etcd підтримує автоматичний TLS, а також автентифікацію через клієнтські сертифікати як для клієнтів до сервера, так і для звʼязку між вузлами (сервер до сервера / кластер). **Зверніть увагу, що etcd стандартно не вмикає [автентифікацію на основі RBAC][auth] або функцію автентифікації на транспортному рівні, щоб зменшити труднощі для користувачів, які починають працювати з базою даних. Крім того, зміна цього стандартного параметра буде порушенням для проєкту, який був заснований у 2013 році. Кластер etcd, який не вмикає функції безпеки, може експонувати свої дані будь-яким клієнтам.**

Щоб почати роботу, спочатку отримайте сертифікат CA та підписану пару ключів для одного учасника. Рекомендується створювати та підписувати нову пару ключів для кожного учасника в кластері.

Для зручності, інструмент [cfssl] надає простий інтерфейс для генерації сертифікатів, і ми надаємо приклад використання інструменту [тут][tls-setup]. Альтернативно, спробуйте цей [посібник з генерації самопідписних пар ключів][tls-guide].

## Основні налаштування {#basic-setup}

etcd приймає кілька параметрів конфігурації, повʼязаних із сертифікатами, або через командні прапорці, або через змінні середовища:

**Звʼязок клієнт-сервер:**

`--cert-file=<path>`: Сертифікат, що використовується для SSL/TLS-зʼєднань **до** etcd. Коли цей параметр встановлено, advertise-client-urls може використовувати схему HTTPS.

`--key-file=<path>`: Ключ для сертифіката. Повинен бути незашифрованим.

`--client-cert-auth`: Коли цей параметр встановлено, etcd перевірятиме всі вхідні HTTPS-запити на наявність клієнтського сертифіката, підписаного довіреним CA, запити, які не надають дійсний клієнтський сертифікат, будуть відхилені. Якщо [автентифікація][auth] увімкнена, сертифікат надає облікові дані для імені користувача, вказаного в полі Common Name.

`--trusted-ca-file=<path>`: Довірений центр сертифікації.

`--auto-tls`: Використовуйте автоматично згенеровані самопідписні сертифікати для TLS-зʼєднань з клієнтами.

**Звʼязок між вузлами (сервер до сервера / кластер):**

Параметри для вузлів працюють так само як і параметри для звʼязку клієнт-сервер:

`--peer-cert-file=<path>`: Сертифікат, що використовується для SSL/TLS-зʼєднань між вузлами. Він буде використовуватися як для прослуховування на адресі вузла, так і для надсилання запитів до інших вузлів.

`--peer-key-file=<path>`: Ключ для сертифіката. Повинен бути незашифрованим.

`--peer-client-cert-auth`: Коли цей параметр встановлено, etcd перевірятиме всі вхідні запити від вузлів кластера на наявність дійсних клієнтських сертифікатів, підписаних наданим CA.

`--peer-trusted-ca-file=<path>`: Довірений центр сертифікації.

`--peer-auto-tls`: Використовуйте автоматично згенеровані самопідписні сертифікати для TLS-зʼєднань між вузлами.

Якщо надано сертифікат для звʼязку клієнт-сервер або вузлів, ключ також повинен бути встановлений. Усі ці параметри конфігурації також доступні через змінні середовища, такі як `ETCD_CA_FILE`, `ETCD_PEER_CA_FILE` тощо.

**Загальні параметри:**

`--cipher-suites`: Список підтримуваних наборів шифрів TLS між сервером/клієнтом і вузлами, розділений комами (порожній буде автоматично заповнений Go).

`--tls-min-version=<version>` Встановлює мінімальну версію TLS, підтримувану etcd.

`--tls-max-version=<version>` Встановлює максимальну версію TLS, підтримувану etcd. Якщо не встановлено, буде використана максимальна версія, підтримувана Go.

## Приклад 1: Транспортна безпека клієнт-сервер з HTTPS {#example-1-client-to-server-transport-security-with-https}

Для цього підготуйте сертифікат CA (`ca.crt`) та підписану пару ключів (`server.crt`, `server.key`).

Налаштуємо etcd для забезпечення простої транспортної безпеки HTTPS крок за кроком:

```sh
$ etcd --name infra0 --data-dir infra0 \
  --cert-file=/path/to/server.crt --key-file=/path/to/server.key \
  --advertise-client-urls=https://127.0.0.1:2379 --listen-client-urls=https://127.0.0.1:2379
```

Це має запуститися без проблем, і можна буде перевірити конфігурацію, використовуючи HTTPS для звʼязку з etcd:

```sh
$ curl --cacert /path/to/ca.crt https://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -v
```

Команда повинна показати, що рукостискання пройшло успішно. Оскільки ми використовуємо самопідписні сертифікати з нашим власним центром сертифікації, CA повинен бути переданий curl за допомогою опції `--cacert`. Інший варіант — додати сертифікат CA до теки довірених сертифікатів системи (зазвичай у `/etc/pki/tls/certs` або `/etc/ssl/certs`).

**Користувачі OSX 10.9+**: curl 7.30.0 на OSX 10.9+ не розуміє сертифікати, передані в командному рядку. Замість цього імпортуйте фіктивний ca.crt безпосередньо в keychain або додайте прапорець `-k` до curl, щоб ігнорувати помилки. Щоб перевірити без прапорця `-k`, запустіть `open ./tests/fixtures/ca/ca.crt` і дотримуйтесь підказок. Будь ласка, видаліть цей сертифікат після тестування! Якщо є обхідний шлях, повідомте нам.

## Приклад 2: Автентифікація клієнт-сервер з клієнтськими сертифікатами HTTPS {#example-2-client-to-server-authentication-with-https-client-certificates}

На цю мить ми надали клієнту etcd можливість перевіряти ідентичність сервера та забезпечувати транспортну безпеку. Однак ми також можемо використовувати клієнтські сертифікати для запобігання несанкціонованого доступу до etcd.

Клієнти надаватимуть свої сертифікати серверу, і сервер перевірятиме, чи підписаний сертифікат наданим CA, і вирішуватиме, чи обслуговувати запит.

Для цього потрібні ті ж файли, що й у першому прикладі, а також пара ключів для клієнта (`client.crt`, `client.key`), підписана тим самим сертифікаційним центром.

```sh
$ etcd --name infra0 --data-dir infra0 \
  --client-cert-auth --trusted-ca-file=/path/to/ca.crt --cert-file=/path/to/server.crt --key-file=/path/to/server.key \
  --advertise-client-urls https://127.0.0.1:2379 --listen-client-urls https://127.0.0.1:2379
```

Тепер спробуйте той самий запит до цього сервера:

```sh
$ curl --cacert /path/to/ca.crt https://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -v
```

Запит має бути відхилений сервером:

```none
...
routines:SSL3_READ_BYTES:sslv3 alert bad certificate
...
```

Щоб зробити його успішним, нам потрібно надати серверу клієнтський сертифікат, підписаний CA:

```sh
$ curl --cacert /path/to/ca.crt --cert /path/to/client.crt --key /path/to/client.key \
  -L https://127.0.0.1:2379/v2/keys/foo -XPUT -d value=bar -v
```

Вихідні дані повинні включати:

```none
...
SSLv3, TLS handshake, CERT verify (15):
...
TLS handshake, Finished (20)
```

А також відповідь від сервера:

```json
{
    "action": "set",
    "node": {
        "createdIndex": 12,
        "key": "/foo",
        "modifiedIndex": 12,
        "value": "bar"
    }
}
```

Вкажіть набори шифрів, щоб заблокувати [слабкі набори шифрів TLS](https://github.com/etcd-io/etcd/issues/8320).

Рукостискання TLS не вдасться, коли клієнтський привіт запитується з недійсними наборами шифрів.

Наприклад:

```bash
$ etcd \
  --cert-file ./server.crt \
  --key-file ./server.key \
  --trusted-ca-file ./ca.crt \
  --cipher-suites TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
```

Тоді клієнтські запити повинні вказувати один із наборів шифрів, зазначених на сервері:

```bash
# дійсний набір шифрів
$ curl \
  --cacert /path/to/ca.crt \
  --cert /path/to/client.crt \
  --key /path/to/client.key \
  -L [CLIENT-URL]/metrics \
  --ciphers ECDHE-RSA-AES128-GCM-SHA256

# запит успішний
etcd_server_version{server_version="3.2.22"} 1
...
```

```bash
# недійсний набір шифрів
$ curl \
  --cacert /path/to/ca.crt \
  --cert /path/to/client.crt \
  --key /path/to/client.key \
  -L [CLIENT-URL]/metrics \
  --ciphers ECDHE-RSA-DES-CBC3-SHA

# запит не вдається з
(35) error:14094410:SSL routines:ssl3_read_bytes:sslv3 alert handshake failure
```

## Приклад 3: Транспортна безпека та клієнтські сертифікати в кластері {#example-3-transport-security-client-certificates-in-a-cluster}

etcd підтримує ту ж модель, що й вище, для **звʼязку між вузлами**, тобто звʼязку між учасниками etcd у кластері.

Припустимо, у нас є наш `ca.crt` і два учасники з власними парами ключів (`member1.crt` і `member1.key`, `member2.crt` і `member2.key`), підписаними цим CA, ми запускаємо etcd наступним чином:

```sh
DISCOVERY_URL=... # з https://discovery.etcd.io/new

# учасник1
$ etcd --name infra1 --data-dir infra1 \
  --peer-client-cert-auth --peer-trusted-ca-file=/path/to/ca.crt --peer-cert-file=/path/to/member1.crt --peer-key-file=/path/to/member1.key \
  --initial-advertise-peer-urls=https://10.0.1.10:2380 --listen-peer-urls=https://10.0.1.10:2380 \
  --discovery ${DISCOVERY_URL}

# учасник2
$ etcd --name infra2 --data-dir infra2 \
  --peer-client-cert-auth --peer-trusted-ca-file=/path/to/ca.crt --peer-cert-file=/path/to/member2.crt --peer-key-file=/path/to/member2.key \
  --initial-advertise-peer-urls=https://10.0.1.11:2380 --listen-peer-urls=https://10.0.1.11:2380 \
  --discovery ${DISCOVERY_URL}
```

Учасники etcd утворять кластер, і весь звʼязок між учасниками в кластері буде зашифрований і автентифікований за допомогою клієнтських сертифікатів. Вихідні дані etcd покажуть, що адреси, до яких він підключається, використовують HTTPS.

## Приклад 4: Автоматична самопідписна транспортна безпека {#example-4-automatic-self-signed-transport-security}

**ПРИМІТКА:** Коли ви вказуєте ClientAutoTLS і PeerAutoTLS, термін дії клієнтського сертифіката та сертифіката вузла, автоматично згенерованого etcd, становить лише 1 рік. Ви можете вказати прапорець --self-signed-cert-validity, щоб встановити термін дії сертифіката в роках.

Для випадків, коли потрібне шифрування звʼязку, але не автентифікація, etcd підтримує шифрування своїх повідомлень за допомогою автоматично згенерованих самопідписних сертифікатів. Це спрощує розгортання, оскільки немає потреби керувати сертифікатами та ключами поза etcd.
Налаштуйте etcd для використання самопідписних сертифікатів для клієнтських і вузлових зʼєднань за допомогою прапорців `--auto-tls` і `--peer-auto-tls`:

```sh
DISCOVERY_URL=... # з https://discovery.etcd.io/new

# учасник1
$ etcd --name infra1 --data-dir infra1 \
  --auto-tls --peer-auto-tls \
  --initial-advertise-peer-urls=https://10.0.1.10:2380 --listen-peer-urls=https://10.0.1.10:2380 \
  --discovery ${DISCOVERY_URL}

# учасник2
$ etcd --name infra2 --data-dir infra2 \
  --auto-tls --peer-auto-tls \
  --initial-advertise-peer-urls=https://10.0.1.11:2380 --listen-peer-urls=https://10.0.1.11:2380 \
  --discovery ${DISCOVERY_URL}
```

Самопідписні сертифікати не автентифікують ідентичність, тому curl поверне помилку:

```sh
curl: (60) SSL certificate problem: Invalid certificate chain
```

Щоб вимкнути перевірку ланцюга сертифікатів, викличте curl з прапорцем `-k`:

```sh
$ curl -k https://127.0.0.1:2379/v2/keys/foo -Xput -d value=bar -v
```

## Примітки для DNS SRV {#notes-for-dns-srv}

Починаючи з v3.1.0 (крім v3.2.9), завантаження SRV для виявлення автентифікує `ServerName` з кореневим доменним імʼям з прапорця `--discovery-srv`. Це робиться для уникнення атак з підміною сертифікатів, вимагаючи, щоб сертифікат мав відповідне кореневе доменне імʼя в полі Subject Alternative Name (SAN). Наприклад, `etcd --discovery-srv=etcd.local` буде автентифікувати вузли/клієнтів лише тоді, коли надані сертифікати мають кореневе доменне імʼя `etcd.local` як запис у полі Subject Alternative Name (SAN).

## Примітки для проксі etcd {#notes-for-etcd-proxy}

Проксі etcd завершує TLS-зʼєднання від свого клієнта, якщо зʼєднання захищене, і використовує власний ключ/сертифікат проксі, вказаний у `--peer-key-file` і `--peer-cert-file`, для звʼязку з учасниками etcd.

Проксі звʼязується з учасниками etcd через `--advertise-client-urls` і `--advertise-peer-urls` даного учасника. Він пересилає клієнтські запити на оголошені клієнтські URL-адреси учасників etcd і синхронізує початкову конфігурацію кластера через оголошені URL-адреси вузлів учасників etcd.

Коли для учасника etcd увімкнено автентифікацію клієнта, адміністратор повинен переконатися, що сертифікат вузла, вказаний у параметрі `--peer-cert-file` проксі, дійсний для цієї автентифікації. Сертифікат вузла проксі також повинен бути дійсним для автентифікації вузла, якщо автентифікація вузла увімкнена.

## Примітки для автентифікації TLS {#notes-for-tls-authentication}

Починаючи з [v3.2.0](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.2.md#v320-2017-06-09), [сертифікати TLS перезавантажуються при кожному клієнтському зʼєднанні](https://github.com/etcd-io/etcd/pull/7829). Це корисно при заміні сертифікатів, що закінчилися, без зупинки серверів etcd; це можна зробити, перезаписавши старі сертифікати новими. Оновлення сертифікатів для кожного зʼєднання не повинно мати великого навантаження, але може бути покращено в майбутньому за допомогою кешування. Приклади тестів можна знайти [тут](https://github.com/etcd-io/etcd/blob/b041ce5d514a4b4aaeefbffb008f0c7570a18986/integration/v3_grpc_test.go#L1601-L1757).

Починаючи з [v3.2.0](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.2.md#v320-2017-06-09), [сервер відхиляє вхідні сертифікати вузлів з неправильним IP `SAN`](https://github.com/etcd-io/etcd/pull/7687). Наприклад, якщо сертифікат вузла містить будь-які IP-адреси в полі Subject Alternative Name (SAN), сервер автентифікує вузол лише тоді, коли віддалена IP-адреса збігається з однією з цих IP-адрес. Це робиться для запобігання несанкціонованим точкам доступу приєднуватися до кластера. Наприклад, CSR вузла B (з `cfssl`) виглядає так:

```json
{
  "CN": "etcd peer",
  "hosts": [
    "*.example.default.svc",
    "*.example.default.svc.cluster.local",
    "10.138.0.27"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "CA",
      "ST": "San Francisco"
    }
  ]
}
```

коли фактична IP-адреса вузла B — `10.138.0.2`, а не `10.138.0.27`. Коли вузол B намагається приєднатися до кластера, вузол A відхилить B з помилкою `x509: certificate is valid for 10.138.0.27, not 10.138.0.2`, оскільки віддалена IP-адреса B не збігається з тією, що в полі Subject Alternative Name (SAN).

Починаючи з [v3.2.0](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.2.md#v320-2017-06-09), [сервер розвʼязує TLS `DNSNames` при перевірці `SAN`](https://github.com/etcd-io/etcd/pull/7767). Наприклад, якщо сертифікат вузла містить лише DNS-імена (без IP-адрес) в полі Subject Alternative Name (SAN), сервер автентифікує вузол лише тоді, коли прямі запити (`dig b.com`) на ці DNS-імена мають відповідну IP-адресу з віддаленою IP-адресою. Наприклад, CSR вузла B (з `cfssl`) виглядає так:

```json
{
  "CN": "etcd peer",
  "hosts": [
    "b.com"
  ],
```

коли віддалена IP-адреса вузла B — `10.138.0.2`. Коли вузол B намагається приєднатися до кластера, вузол A шукає вхідний хост `b.com`, щоб отримати список IP-адрес (наприклад, `dig b.com`). І відхиляє B, якщо список не містить IP-адресу `10.138.0.2`, з помилкою `tls: 10.138.0.2 does not match any of DNSNames ["b.com"]`.

Починаючи з [v3.2.2](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.2.md#v322-2017-07-07), [сервер приймає зʼєднання, якщо IP-адреса збігається, без перевірки DNS-записів](https://github.com/etcd-io/etcd/pull/8223). Наприклад, якщо сертифікат вузла містить IP-адреси та DNS-імена в полі Subject Alternative Name (SAN), і віддалена IP-адреса збігається з однією з цих IP-адрес, сервер просто приймає зʼєднання без подальшої перевірки DNS-імен. Наприклад, CSR вузла B (з `cfssl`) виглядає так:

```json
{
  "CN": "etcd peer",
  "hosts": [
    "invalid.domain",
    "10.138.0.2"
  ],
```

коли віддалена IP-адреса вузла B — `10.138.0.2`, а `invalid.domain` — недійсний хост. Коли вузол B намагається приєднатися до кластера, вузол A успішно автентифікує B, оскільки поле Subject Alternative Name (SAN) містить дійсну відповідну IP-адресу. Див. [issue#8206](https://github.com/etcd-io/etcd/issues/8206) для отримання додаткової інформації.

Починаючи з [v3.2.5](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.2.md#v325-2017-08-04), [сервер підтримує зворотний пошук на основі підстановочних знаків DNS `SAN`](https://github.com/etcd-io/etcd/pull/8281). Наприклад, якщо сертифікат вузла містить лише DNS-імена (без IP-адрес) в полі Subject Alternative Name (SAN), сервер спочатку виконує зворотний пошук віддаленої IP-адреси, щоб отримати список імен, що відповідають цій адресі (наприклад, `nslookup IPADDR`). Потім приймає зʼєднання, якщо ці імена мають відповідне імʼя з DNS-іменами сертифіката вузла (або за точним або підстановочним збігом). Якщо жоден зворотний/прямий пошук не спрацював, він повертає помилку `"tls: "10.138.0.2" does not match any of DNSNames ["*.example.default.svc","*.example.default.svc.cluster.local"]`. Див. [issue#8268](https://github.com/etcd-io/etcd/issues/8268) для отримання додаткової інформації.

[v3.3.0](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.3.md) додає прапорець [`etcd --peer-cert-allowed-cn`](https://github.com/etcd-io/etcd/pull/8616) для підтримки [автентифікації на основі CN (Common Name) для міжвузлових зʼєднань](https://github.com/etcd-io/etcd/issues/8262). TLS-автентифікація Kubernetes включає генерацію динамічних сертифікатів для учасників etcd та інших системних компонентів (наприклад, API-сервер, kubelet тощо). Підтримка різних сертифікаційних центрів для кожного компонента забезпечує більш жорсткий контроль доступу до кластера etcd, але часто є трудомісткою. Коли вказано прапорець `--peer-cert-allowed-cn`, вузол може приєднатися лише з відповідним загальним імʼям, навіть зі спільними центрами сертифікації. Наприклад, кожен учасник у кластері з 3 вузлів налаштований з CSR (з `cfssl`) наступним чином:

```json
{
  "CN": "etcd.local",
  "hosts": [
    "m1.etcd.local",
    "127.0.0.1",
    "localhost"
  ],
```

```json
{
  "CN": "etcd.local",
  "hosts": [
    "m2.etcd.local",
    "127.0.0.1",
    "localhost"
  ],
```

```json
{
  "CN": "etcd.local",
  "hosts": [
    "m3.etcd.local",
    "127.0.0.1",
    "localhost"
  ],
```

Тоді лише вузли з відповідними загальними іменами будуть автентифіковані, якщо вказано прапорець `--peer-cert-allowed-cn etcd.local`. І вузли з різними CN у CSR або різними `--peer-cert-allowed-cn` будуть відхилені:

```bash
$ etcd --peer-cert-allowed-cn m1.etcd.local

I | embed: rejected connection from "127.0.0.1:48044" (error "CommonName authentication failed", ServerName "m1.etcd.local")
I | embed: rejected connection from "127.0.0.1:55702" (error "remote error: tls: bad certificate", ServerName "m3.etcd.local")
```

Кожен процес повинен бути запущений з:

```bash
etcd --peer-cert-allowed-cn etcd.local

I | pkg/netutil: resolving m3.etcd.local:32380 to 127.0.0.1:32380
I | pkg/netutil: resolving m2.etcd.local:22380 to 127.0.0.1:22380
I | pkg/netutil: resolving m1.etcd.local:2380 to 127.0.0.1:2380
I | etcdserver: published {Name:m3 ClientURLs:[https://m3.etcd.local:32379]} to cluster 9db03f09b20de32b
I | embed: ready to serve client requests
I | etcdserver: published {Name:m1 ClientURLs:[https://m1.etcd.local:2379]} to cluster 9db03f09b20de32b
I | embed: ready to serve client requests
I | etcdserver: published {Name:m2 ClientURLs:[https://m2.etcd.local:22379]} to cluster 9db03f09b20de32b
I | embed: ready to serve client requests
I | embed: serving client requests on 127.0.0.1:32379
I | embed: serving client requests on 127.0.0.1:22379
I | embed: serving client requests on 127.0.0.1:2379
```

[v3.2.19](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.2.md) і [v3.3.4](https://github.com/etcd-io/etcd/blob/master/CHANGELOG-3.3.md) виправляють перезавантаження TLS, коли [поле SAN сертифіката містить лише IP-адреси, але не доменні імена](https://github.com/etcd-io/etcd/issues/9541). Наприклад, учасник налаштований з CSR (з `cfssl`) наступним чином:

```json
{
  "CN": "etcd.local",
  "hosts": [
    "127.0.0.1"
  ],
```

У Go сервер викликає `(*tls.Config).GetCertificate` для перезавантаження TLS, якщо і тільки якщо поле `(*tls.Config).Certificates` сервера не порожнє, або поле `(*tls.ClientHelloInfo).ServerName` не порожнє з дійсним SNI від клієнта. Раніше etcd завжди заповнював `(*tls.Config).Certificates` під час початкового клієнтського TLS-рукостискання як непорожнє. Таким чином, від клієнта завжди очікувалося надання відповідного SNI для проходження перевірки TLS і для виклику `(*tls.Config).GetCertificate` для перезавантаження TLS-активів.

Однак сертифікат, чиє поле SAN [не містить жодних доменних імен, а лише IP-адреси](https://github.com/etcd-io/etcd/issues/9541), запитуватиме `*tls.ClientHelloInfo` з порожнім полем `ServerName`, таким чином не викликаючи перезавантаження TLS під час початкового TLS-рукостискання; це стає проблемою, коли потрібно замінити сертифікати, що закінчилися, онлайн.

Тепер `(*tls.Config).Certificates` створюється порожнім під час початкового клієнтського TLS-рукостискання, спочатку для виклику `(*tls.Config).GetCertificate`, а потім для заповнення решти сертифікатів під час кожного нового TLS-зʼєднання, навіть коли клієнтський SNI порожній (наприклад, сертифікат містить лише IP-адреси).

## Примітки для білого списку хостів {#notes-for-host-whitelist}

Прапорець `etcd --host-whitelist` вказує допустимі імена хостів з HTTP-клієнтських запитів. Політика походження клієнта захищає від атак ["DNS Rebinding"](https://en.wikipedia.org/wiki/DNS_rebinding) на незахищені сервери etcd. Тобто будь-який вебсайт може просто створити авторизоване DNS-імʼя і направити DNS на `"localhost"` (або будь-яку іншу адресу). Тоді всі HTTP-точки доступу сервера etcd, що слухають на `"localhost"`, стають доступними, таким чином вразливими до атак DNS Rebinding. Див. [CVE-2018-5702](https://bugs.chromium.org/p/project-zero/issues/detail?id=1447#c2) для отримання додаткової інформації.

Політика походження клієнта працює наступним чином:

1. Якщо клієнтське зʼєднання захищене через HTTPS, дозволяйте будь-які імена хостів.
2. Якщо клієнтське зʼєднання не захищене і `"HostWhitelist"` не порожній, дозволяйте лише HTTP-запити, чиє поле Host вказане в білому списку.

Зверніть увагу, що політика походження клієнта застосовується незалежно від того, чи увімкнена автентифікація, для більш жорсткого контролю.

Стандартно, `etcd --host-whitelist` і `embed.Config.HostWhitelist` встановлені *порожніми*, щоб дозволити всі імена хостів. Зверніть увагу, що при вказанні імен хостів, адреси зворотного звʼязку не додаються автоматично. Щоб дозволити інтерфейси зворотного звʼязку, додайте їх до білого списку вручну (наприклад, `"localhost"`, `"127.0.0.1"` тощо).

## Часті питання {#frequently-asked-questions}

### Я бачу помилку SSLv3 alert handshake failure при використанні автентифікації клієнта TLS? {#im-seeing-a-sslv3-alert-handshake-failure-when-using-tls-client-authentication}

Пакунок `crypto/tls` з `golang` перевіряє використання ключа сертифіката перед його використанням. Щоб використовувати ключ сертифіката для автентифікації клієнта, нам потрібно додати `clientAuth` до `Extended Key Usage` при створенні ключа сертифіката.

Ось як це зробити:

Додайте наступний розділ до openssl.cnf:

```conf
[ ssl_client ]
...
  extendedKeyUsage = clientAuth
...
```

При створенні сертифіката обовʼязково вкажіть його в прапорці `-extensions`:

```sh
$ openssl ca -config openssl.cnf -policy policy_anything -extensions ssl_client -out certs/machine.crt -infiles machine.csr
```

Функція [`Verify`](https://pkg.go.dev/crypto/x509#Certificate.Verify) у логіці `crypto/x509` реалізує загальне, але нестандартне розширення — вона вимагає, щоб сертифікати CA та проміжні сертифікати або не визначали жодного розширеного використання ключа, або надмножину тих, що на кінцевих сертифікатах. Якщо сертифікати у вашому ланцюжку визначають будь-які розширені використання ключа, вони також повинні включати `serverAuth` та/або `clientAuth`.

Інакше ви можете побачити помилку на кшталт `unsuitable certificate purpose` (OpenSSL) або `certificate specifies an incompatible key usage` (Go).

### При автентифікації сертифіката вузла я отримую "certificate is valid for 127.0.0.1, not $MY_IP" {#with-peer-certificate-authentication-i-receive-certificate-is-valid-for-127001-not-my_ip}

Переконайтеся, що сертифікати підписані з іменем Subject Name, що відповідає публічній IP-адресі учасника. Інструмент `etcd-ca`, наприклад, надає опцію `--ip=` для команди `new-cert`.

Сертифікат повинен бути підписаний для повного доменного імені учасника в полі Subject Name, використовуйте Subject Alternative Names (короткі IP SANs) для додавання IP-адреси. Інструмент `etcd-ca` надає опцію `--domain=` для команди `new-cert`, а openssl може зробити [це][alt-name] також.

### Чи шифрує etcd дані, збережені на дисках? {#does-etcd-encrypt-data-stored-on-disk-drives}

Ні. etcd не шифрує дані ключів/значень, збережені на дисках. Якщо користувачеві потрібно шифрувати дані, збережені в etcd, є кілька варіантів:

* Нехай клієнтські застосунки шифрують і дешифрують дані
* Використовуйте функцію підсистем зберігання для шифрування збережених даних, наприклад, [dm-crypt]

### Я бачу попередження в журналі, що "тека X існує без рекомендованих дозволів `-rwx------`" {#im-seeing-a-log-warning-that-directory-x-exist-without-recommended-permission--rwx------}

Коли etcd створює певні нові теки, він встановлює дозволи файлів на 700, щоб запобігти несанкціонованому доступу, наскільки це можливо. Однак, якщо користувач вже створив теку за власним уподобанням, etcd використовує наявну теку і записує попередження, якщо дозволи відрізняються від 700.

[alt-name]: http://wiki.cacert.org/FAQ/subjectAltName
[auth]: ../authentication/
[cfssl]: https://github.com/cloudflare/cfssl
[dm-crypt]: https://en.wikipedia.org/wiki/Dm-crypt
[tls-guide]: https://github.com/coreos/docs/blob/master/os/generate-self-signed-certificates.md
[tls-setup]: https://github.com/etcd-io/etcd/tree/master/hack/tls-setup

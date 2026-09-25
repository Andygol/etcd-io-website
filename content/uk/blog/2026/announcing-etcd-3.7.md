---
title: Повідомлення про випуск etcd v3.7.0
author:  "SIG-Etcd Leads"
date: 2026-07-08
draft: false
---

## Зміст {#table-of-contents}

- [Вступ](#introduction)
- [Основні функції](#major-features)
- [Функції](#features)
  - [RangeStream](#rangestream)
  - [Покращення продуктивності](#performance-improvements)
    - [Оптимізація запитів лише з ключами](#keys-only-range-optimization)
    - [Швидші та надійніші оренди etcd](#faster-more-reliable-etcd-leases)
    - [Швидші операції find()](#faster-find-operations)
  - [Інші функції](#other-features)
    - [Модернізація protobuf](#protobuf-overhaul)
    - [Підтримка Unix-сокетів](#unix-socket-support)
    - [Завантаження з v3store](#bootstrap-from-v3store)
    - [Тайм-аути etcdutl](#etcdutl-timeouts)
    - [Пряме встановлення токена автентифікації](#setting-the-authentication-token-directly)
    - [Отримання AuthStatus без автентифікації](#retrieve-authstatus-without-authenticating)
    - [Нові метрики спостереження](#new-watch-metrics)
    - [Очищення команд etcdctl](#etcdctl-command-cleanup)
- [Оновлення](#upgrading)
  - [Видалення експериментальних прапорців](#experimental-flags-removed)
  - [Очищення пакунків та коду API Legacy V2](#legacy-v2-api-packages-and-code-cleanup)
  - [Неблокуюче створення клієнта](#non-blocking-client-creation)
  - [Лише багатоархітектурні контейнерні образи](#multiarch-container-images-only)
  - [Зміни в API](#api-changes)
- [bbolt v1.5.1](#bbolt-v151)
- [raft v3.7.0](#raft-v370)
- [Оновлення залежностей](#dependency-updates)
- [Учасники](#contributors)
  - [Лідери](#leads)
  - [Інші учасники](#other-contributors)
  - [Нові учасники](#new-contributors)

## Вступ {#introduction}

Сьогодні SIG etcd випускає [etcd v3.7.0](https://github.com/etcd-io/etcd/releases/tag/v3.7.0) — останній мінорний випуск популярного розподіленого сховища ключ-значення та ключового компонента Kubernetes. v3.7 містить довгоочікувану функцію RangeStream, надає кілька інших покращень продуктивності, видаляє останні залишки застарілого v2store та завершує масштабну модернізацію protobuf.

Ви можете завантажити etcd v3.7.0 тут:

- [Вихідний код](https://github.com/etcd-io/etcd/archive/refs/tags/v3.7.0.tar.gz)
- [Бінарні файли](https://github.com/etcd-io/etcd/releases/tag/v3.7.0)
- [Офіційні контейнерні образи](https://gcr.io/etcd-development/etcd)

Цей випуск також містить нові версії двох основних залежностей etcd — [bbolt v1.5.0](https://github.com/etcd-io/bbolt/releases/tag/v1.5.0) та [raft v3.7.0](https://github.com/etcd-io/raft/releases/tag/v3.7.0).

Інструкції зі встановлення etcd дивіться в [документації зі встановлення](/docs/v3.7/install/). Повний перелік змін дивіться в [журналі змін etcd v3.7](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.7.md).

Щиро дякуємо всім учасникам, які зробили цей випуск можливим!

## Основні функції {#major-features}

Найсуттєвіші зміни у v3.7.0 включають:

- [**RangeStream**](#rangestream) — потокова передача великих наборів результатів частинами замість буферизації всієї відповіді.
- **Запити лише з ключами, швидші та надійніші оренди** та кілька інших [**покращень продуктивності**](#performance-improvements).
- etcd тепер [повністю завантажується з v3store](#bootstrap-from-v3store), усуваючи давню залежність від застарілого v2 store.
- Завершена [**модернізація protobuf**](#protobuf-overhaul), яка замінює застарілі бібліотеки protobuf на повністю підтримувані.
- etcd v3.7 постачається з [bbolt v1.5.1](#bbolt-v151) та [raft v3.7.0](#raft-v370).

## Функції {#features}

### RangeStream {#rangestream}

У etcd v3.6 та раніших версіях складно працювати із запитами, що повертають великі набори результатів. База даних буферизувала весь набір результатів перед надсиланням, що призводило до непередбачуваної затримки та використання памʼяті як на сервері, так і на клієнті. [RPC RangeStream](https://github.com/kubernetes/enhancements/tree/master/keps/sig-etcd/5966-etcd-range-stream) дозволяє застосункам, що викликають, приймати набори результатів частинами, зменшуючи затримку та роблячи використання памʼяті для буферизації більш передбачуваним.

Інструкції щодо використання RangeStream [у викликах gRPC](/docs/v3.7/learning/api/#rangestream) та [в etcdctl](/docs/v3.7/dev-guide/interacting_v3/#read-keys) можна знайти в документації etcd. Користувачам варто спробувати цю функцію у власних застосунках.

У скоординованих випусках функція RangeStream стане доступною користувачам, які працюють із майбутнім v1.37 Kubernetes, після ввімкнення функціональної можливості `EtcdRangeStream`. Таке раннє та заплановане впровадження стало можливим завдяки обʼєднанню розробки etcd та Kubernetes у 2023 році.

### Покращення продуктивності {#performance-improvements}

v3.7 надає кілька конкретних покращень продуктивності — як для панелі управління Kubernetes, так і для інших сценаріїв використання. Користувачі Kubernetes мають побачити значне зниження загального використання CPU членами etcd порівняно з v3.6.

#### Оптимізація запитів лише з ключами {#keys-only-range-optimization}

etcd v3.7.0 містить оптимізацію Range лише з ключами ([#21791: keys-only Range optimization](https://github.com/etcd-io/etcd/pull/21791)). Під час обробки запиту Range з `keys_only` або `etcdctl get --keys-only` etcd читає виключно зі свого індексу в памʼяті. Він повертає відповідні ключі без завантаження всіх серіалізованих значень із bbolt, як це було раніше. Єдиний виняток, коли завантаження з bbolt все ще потрібне, — це коли запити Range з `keys_only` мають бути відсортовані за значенням (тобто коли SortTarget встановлено у VALUE).

Це зменшує зайві звернення до сховища та використання памʼяті для робочих навантажень, яким потрібні лише імена ключів, роблячи великі запити лише з ключами більш ефективними.

#### Швидші та надійніші оренди etcd {#faster-more-reliable-etcd-leases}

v3.7 покращує закінчення терміну дії та оновлення оренди:

- Запити LeaseRevoke тепер пріоритезуються, щоб забезпечити своєчасне закінчення терміну дії оренди під час перевантаження ([#20492: stability enhancement during overload conditions](https://github.com/etcd-io/etcd/pull/20492)).
- Нова функція FastLeaseKeepAlive забезпечує швидше оновлення оренди, пропускаючи очікування застосованого індексу ([#20589: etcdserver: improve linearizable renew lease](https://github.com/etcd-io/etcd/pull/20589)).

#### Швидші операції find() {#faster-find-operations}

etcd 3.7 покращує продуктивність одночасних спостережень за ключами, роблячи операції find() швидшими ([#19768: adt: split interval tree by right endpoint on matched left endpoints](https://github.com/etcd-io/etcd/pull/19768)).

### Інші функції {#other-features}

#### Модернізація protobuf {#protobuf-overhaul}

v3.7 мігрує та замінює кілька застарілих бібліотек protobuf на повністю підтримувані залежності. Це включає заміну `github.com/golang/protobuf` та `github.com/gogo/protobuf` на повністю підтримуваний `google.golang.org/protobuf` ([#14533: Protobuf: cleanup both golang/protobuf and gogo/protobuf](https://github.com/etcd-io/etcd/issues/14533)), а також міграцію grpc-logging на grpc-middleware v2 ([#20420: Migrate grpc-logging to grpc-middleware v2](https://github.com/etcd-io/etcd/pull/20420)).

Окрім покращення безпеки та зручності супроводу, цей рефакторинг, як було показано, зменшує використання CPU компонентами etcd.

Хоча ці зміни не мають безпосередньо вплинути на користувачів, які запускають etcd через офіційні бінарні файли або контейнерні образи, вони можуть вплинути на користувачів, які залежать від Go-модулів etcd, таких як клієнтський SDK або пакунки в api/ чи pkg/. Таким споживачам може знадобитися оновити свій код або залежності через зміни protobuf та повʼязані зміни в API, внесені в цьому випуску. Детальнішу інформацію можна знайти в [тікеті відстеження змін в API](https://github.com/etcd-io/website/issues/1162).

#### Підтримка Unix-сокетів {#unix-socket-support}

etcd тепер підтримує точки доступу Unix-сокетів ([#19760: Add Support for Unix Socket endpoints](https://github.com/etcd-io/etcd/pull/19760)), що дозволяє локальну комунікацію без TCP-порту. Оскільки це обмежено одноелементними кластерами, ця функція в основному призначена для розробки, тестування та сценаріїв використання на периферійних пристроях.

#### Завантаження з v3store {#bootstrap-from-v3store}

Одна з головних змін в etcd v3.7 полягає в тому, що сервер тепер повністю завантажується з v3 store ([#20187 Bootstrap etcdserver from v3store](https://github.com/etcd-io/etcd/issues/20187)), усуваючи залежність від застарілого v2 store під час запуску.

Ця віха є результатом довгострокових зусиль, що охоплюють кілька випусків — від v3.4 до v3.7. Вона усуває давній технічний борг, значно спрощує процес завантаження та закладає основу для майбутніх покращень etcd.

Щоб зберегти зворотну сумісність, etcd v3.7 продовжує генерувати v2-знімки. Відповідно, прапорець `--snapshot-count` також збережений у v3.7. Це остання залежність від застарілого v2 store, і як генерація v2-знімків, так і прапорець `--snapshot-count` будуть видалені у v3.8.

#### Тайм-аути etcdutl {#etcdutl-timeouts}

Усі команди etcdutl тепер мають аргумент командного рядка для тайм-ауту ([#20708: etcdutl: enable timeout functionality for all commands](https://github.com/etcd-io/etcd/pull/20708)), тому офлайн-команди утиліт більше не блокуються на невизначений час, утримуючи блокування.

#### Пряме встановлення токена автентифікації {#setting-the-authentication-token-directly}

Клієнт v3 тепер дозволяє користувачам встановлювати JWT безпосередньо, надаючи більше гнучкості у варіантах автентифікації ([#16803: clientv3: allow setting JWT directly](https://github.com/etcd-io/etcd/pull/16803), [#20747: clientv3: disable auth retry when token is set](https://github.com/etcd-io/etcd/pull/20747)).

#### Отримання AuthStatus без автентифікації {#retrieve-authstatus-without-authenticating}

Клієнти можуть перевіряти свій AuthStatus без попередньої спроби автентифікації, усуваючи частину накладних витрат застосунку ([#20802: etcdserver: remove permission check on AuthStatus api](https://github.com/etcd-io/etcd/pull/20802)).

#### Нові метрики спостереження {#new-watch-metrics}

v3.7 додає опційні метрики циклу надсилання спостережень ([\#21030: Instrument watchstream send loop](https://github.com/etcd-io/etcd/pull/21030)) для кращої спостережуваності шляху спостереження:

- `etcd_debugging_server_watch_send_loop_watch_stream_duration_seconds`
- `etcd_debugging_server_watch_send_loop_watch_stream_duration_per_event_seconds`
- `etcd_debugging_server_watch_send_loop_control_stream_duration_seconds`
- `etcd_debugging_server_watch_send_loop_progress_duration_seconds`

Також зʼявилася нова метрика etcd_server_request_duration_seconds ([#21038: Add metric etcd_server_request_duration_seconds](https://github.com/etcd-io/etcd/pull/21038)).

#### Очищення команд etcdctl {#etcdctl-command-cleanup}

Команди etcdctl були реорганізовані для зрозумілості ([#20162: etcdctl: organize etcdctl subcommand](https://github.com/etcd-io/etcd/pull/20162)), а глобальні аргументи командного рядка тепер приховані, щоб спростити вивід довідки ([#20493: etcdctl: hide global flags](https://github.com/etcd-io/etcd/pull/20493)).

## Оновлення {#upgrading}

Цей випуск містить зміни, що порушують сумісність, особливо навколо видалення застарілих компонентів v2. Користувачам слід ознайомитися з [посібником з оновлення](/docs/v3.7/upgrades/upgrade_3_7/) перед оновленням своїх вузлів. Як і для всіх мінорних випусків, виконуйте поступове оновлення по одному члену за раз і підтверджуйте стан справності кластера між кроками.

### Видалення експериментальних прапорців {#experimental-flags-removed}

Усі застарілі експериментальні прапорці видалено ([#19959: Cleanup the deprecated experimental flags](https://github.com/etcd-io/etcd/pull/19959)). Функції в etcd тепер дотримуються Kubernetes-стилі життєвого циклу функціональних можливостей (Alpha → Beta → GA), запровадженого у v3.6, замість старого префікса `--experimental`. Якщо ваша конфігурація досі покладається на аргументи командного рядка `--experimental-*`, мігруйте на відповідні функціональні можливості або стабільні аргументи командного рядка перед оновленням до etcd 3.7.

### Очищення пакунків та коду API Legacy V2 {#legacy-v2-api-packages-and-code-cleanup}

Щоб усунути залежності від v2store, було видалено такі компоненти:

- [v2 discovery](https://github.com/etcd-io/etcd/pull/20109) ([#20109: Remove v2discovery](https://github.com/etcd-io/etcd/pull/20109)) — пакунки видалено,
- підтримку [v2 request](https://github.com/etcd-io/etcd/pull/21263) ([#21263: Remove v2 Request and apply_v2.go](https://github.com/etcd-io/etcd/pull/21263)),
- підтримку [v2 client](https://github.com/etcd-io/etcd/pull/20117) ([#20117: Remove client/internal/v2](https://github.com/etcd-io/etcd/pull/20117)).

Ці зміни можуть спричинити певні проблеми для користувачів, особливо для тих, хто ще не оновився до v3.6.11 або новішої версії. Користувачам слід повідомляти про будь-які перешкоди, з якими вони стикаються, або про випадки, що потребують кращої документації з оновлення.

### Неблокуюче створення клієнта {#non-blocking-client-creation}

etcd більше не враховує застарілу опцію діалу `grpc.WithBlock` ([\#21942: Make the etcd client creation non-blocking](https://github.com/etcd-io/etcd/pull/21942)). Щоб зберегти попередню блокуючу поведінку, коли це потрібно, дотримуйтеся вказівок у [документації з антипатернів](https://github.com/grpc/grpc-go/blob/master/Documentation/anti-patterns.md#especially-bad-using-deprecated-dialoptions) grpc-go.

### Лише багатоархітектурні контейнерні образи {#multiarch-container-images-only}

Для користувачів, які покладаються на офіційні контейнерні образи etcd, v3.7 буде поширюватися **лише** як багатоархітектурні контейнери. Образи з тегами архітектур не будуть доступні, тому відповідно налаштуйте свої розгортання.

### Зміни в API {#api-changes}

Як і в кожному випуску etcd, є низка змін в API. Вони розроблені так, щоб бути максимально зворотно сумісними, але можуть вимагати коригування деякими користувачами. Повну інформацію дивіться на нашій [сторінці документації з API](/docs/v3.7/learning/api/).

## bbolt v1.5.1 {#bbolt-v151}

etcd v3.7 залежить від [v1.5.1](https://github.com/etcd-io/bbolt/blob/main/CHANGELOG/CHANGELOG-1.5.md) рушія зберігання bbolt та включає його. v1.5 містить кілька покращень функціональності та продуктивності, зокрема:

- [Обмеження розміру файлу бази даних](https://github.com/etcd-io/bbolt/pull/929): користувачі можуть встановлювати обмеження розміру файлу, і bbolt їх дотримуватиметься. Коли база даних bolt перевищує ці обмеження, вона відмовлятиметься приймати записи, доки базу даних не буде ущільнено або обмеження не буде змінено.
- [Вимкнення статистики для продуктивності](https://github.com/etcd-io/bbolt/pull/977): користувачі можуть встановити `NoStatistics`, щоб обмежити накладні витрати від блокувань, які бере переглядач статистики бази даних.
- [Ефективніша обробка hashmap](https://github.com/etcd-io/bbolt/pull/1179): швидше злиття діапазонів із меншими накладними витратами.

## raft v3.7.0 {#raft-v370}

etcd 3.7 залежить від v3.7.0 рушія консенсусу raft та включає його. v3.7 містить кілька покращень, зокрема:

- [Оновлення процесу завантаження](https://github.com/etcd-io/raft/pull/370): v3.7 тепер дозволяє завантаження з частково ініціалізованих знімків, підтримуючи безпосередню ініціалізацію etcd з v3store.
- [Покращення потоку ReadIndex для запобігання застарілим читанням](https://github.com/etcd-io/raft/pull/397) шляхом впровадження унікального ідентифікатора в контекст heartbeat для операцій лише для читання.

raft v3.7.0 також містить [ті самі оновлення бібліотек protobuf](https://github.com/etcd-io/etcd/issues/14533) та рефакторинг, що й etcd.

## Оновлення залежностей {#dependency-updates}

Інші оновлення залежностей включають підвищення `golang.org/x/crypto` до v0.52.0 для вирішення CVE ([#21903: \[release-3.7\] Bump golang.org/x/crypto to v0.52.0](https://github.com/etcd-io/etcd/pull/21903)), оновлення OpenTelemetry contrib до v0.61.0 ([#20017: Update otelgrpc to v0.61.0](https://github.com/etcd-io/etcd/pull/20017)) та компіляцію з Go 1.26.4 ([#21891: \[release-3.7\] Update Go to 1.26.4](https://github.com/etcd-io/etcd/pull/21891)).

## Учасники {#contributors}

etcd v3.7.0 — це продукт понад сотні учасників з усієї спільноти. Дякуємо всім, хто писав код, рецензував PR, створював і триажував проблеми та допомагав тестувати альфа-, бета-версії та кандидати на випуск.

### Лідери {#leads}

Лідерами SIG etcd для випуску v3.7 є [ivanvc](https://github.com/ivanvc), [serathius](https://github.com/serathius), [ahrtr](https://github.com/ahrtr), [fuweid](https://github.com/fuweid), [siyuanfoundation](https://github.com/siyuanfoundation) та [jberkus](https://github.com/jberkus). Іван очолює нашу команду випуску.

### Інші учасники {#other-contributors}

[ah8ad3](https://github.com/ah8ad3), [ajaysundark](https://github.com/ajaysundark), [aladesawe](https://github.com/aladesawe), [amosehiguese](https://github.com/amosehiguese), [ArkaSaha30](https://github.com/ArkaSaha30), [ashikjm](https://github.com/ashikjm), [AwesomePatrol](https://github.com/AwesomePatrol), [dims](https://github.com/dims), [Elbehery](https://github.com/Elbehery), [gangli113](https://github.com/gangli113), [henrybear327](https://github.com/henrybear327), [Jille](https://github.com/Jille), [jmhbnz](https://github.com/jmhbnz), [joshjms](https://github.com/joshjms), [joshuazh-x](https://github.com/joshuazh-x), [kishen-v](https://github.com/kishen-v), [lavishpal](https://github.com/lavishpal), [liggitt](https://github.com/liggitt), [marcelfranca](https://github.com/marcelfranca), [miancheng7](https://github.com/miancheng7), [mmorel-35](https://github.com/mmorel-35), [MrDXY](https://github.com/MrDXY), [mrueg](https://github.com/mrueg), [purpleidea](https://github.com/purpleidea), [qsyqian](https://github.com/qsyqian), [redwrasse](https://github.com/redwrasse), [ronaldngounou](https://github.com/ronaldngounou), [skitt](https://github.com/skitt), [spzala](https://github.com/spzala), [tcchawla](https://github.com/tcchawla), [tjungblu](https://github.com/tjungblu), [vivekpatani](https://github.com/vivekpatani), [wenjiaswe](https://github.com/wenjiaswe)

### Нові учасники {#new-contributors}

Особливе вітання учасникам, які зробили свій перший внесок в etcd у цьому циклі — зокрема [Jeffrey Ying](https://github.com/jefftree), чия робота просунула функцію RangeStream. Нові учасники можуть мати суттєвий вплив на etcd; якщо ви хочете долучитися, дивіться [посібник для учасників](https://github.com/etcd-io/etcd/blob/main/CONTRIBUTING.md).

[1911860538](https://github.com/1911860538), [4rivappa](https://github.com/4rivappa), [aaronjzhang](https://github.com/aaronjzhang), [abdurrehman107](https://github.com/abdurrehman107), [ABin-Huang](https://github.com/ABin-Huang), [adeptvin1](https://github.com/adeptvin1), [aditya7880900936](https://github.com/aditya7880900936), [AHBICJ](https://github.com/AHBICJ), [akstron](https://github.com/akstron), [alliasgher](https://github.com/alliasgher), [aman4433](https://github.com/aman4433), [aojea](https://github.com/aojea), [apullo777](https://github.com/apullo777), [AR21SM](https://github.com/AR21SM), [arturmelanchyk](https://github.com/arturmelanchyk), [AshrafAhmed9](https://github.com/AshrafAhmed9), [asttool](https://github.com/asttool), [asutorufa](https://github.com/asutorufa), [BBQing](https://github.com/BBQing), [beforetech](https://github.com/beforetech), [boqishan](https://github.com/boqishan), [caltechustc](https://github.com/caltechustc), [carsontham](https://github.com/carsontham), [christophsj](https://github.com/christophsj), [chuanye-gao](https://github.com/chuanye-gao), [cnuss](https://github.com/cnuss), [cuiweixie](https://github.com/cuiweixie), [dmvolod](https://github.com/dmvolod), [Dogacel](https://github.com/Dogacel), [dongjiang1989](https://github.com/dongjiang1989), [EduardoVega](https://github.com/EduardoVega), [evertrain](https://github.com/evertrain), [eyupcanakman](https://github.com/eyupcanakman), [gaganhr94](https://github.com/gaganhr94), [goingforstudying-ctrl](https://github.com/goingforstudying-ctrl), [greenblade29](https://github.com/greenblade29), [Himanshu-370](https://github.com/Himanshu-370), [HossamSaberX](https://github.com/HossamSaberX), [huajianxiaowanzi](https://github.com/huajianxiaowanzi), [hwdef](https://github.com/hwdef), [ishan-gupta2005](https://github.com/ishan-gupta2005), [ishan16696](https://github.com/ishan16696), [ivangsm](https://github.com/ivangsm), [JasonLove-Coding](https://github.com/JasonLove-Coding), [Jefftree](https://github.com/Jefftree), [jihogh](https://github.com/jihogh), [jonathan-albrecht-ibm](https://github.com/jonathan-albrecht-ibm), [kairosci](https://github.com/kairosci), [kei01234kei](https://github.com/kei01234kei), [kjgorman](https://github.com/kjgorman), [kovan](https://github.com/kovan), [kstrifonoff](https://github.com/kstrifonoff), [Kunalbehbud](https://github.com/Kunalbehbud), [letreturn](https://github.com/letreturn), [lorenz](https://github.com/lorenz), [m4l1c1ou5](https://github.com/m4l1c1ou5), [madhav-murali](https://github.com/madhav-murali), [madvimer](https://github.com/madvimer), [majiayu000](https://github.com/majiayu000), [marcus-hodgson-antithesis](https://github.com/marcus-hodgson-antithesis), [mattsains](https://github.com/mattsains), [mcrute](https://github.com/mcrute), [mingl1](https://github.com/mingl1), [MohanadKh03](https://github.com/MohanadKh03), [mstrYoda](https://github.com/mstrYoda), [NAM-MAN](https://github.com/NAM-MAN), [neeraj542](https://github.com/neeraj542), [nicknikolakakis](https://github.com/nicknikolakakis), [nihalmaddala](https://github.com/nihalmaddala), [niuyueyang1996](https://github.com/niuyueyang1996), [notandruu](https://github.com/notandruu), [ntdkhiem](https://github.com/ntdkhiem), [nwnt](https://github.com/nwnt), [olamilekan000](https://github.com/olamilekan000), [pigeio](https://github.com/pigeio), [pjsharath28](https://github.com/pjsharath28), [progmem](https://github.com/progmem), [Qian-Cheng-nju](https://github.com/Qian-Cheng-nju), [quocvibui](https://github.com/quocvibui), [ravisastryk](https://github.com/ravisastryk), [robin-vidal](https://github.com/robin-vidal), [robinkb](https://github.com/robinkb), [rockswe](https://github.com/rockswe), [roman-khimov](https://github.com/roman-khimov), [rsafonseca](https://github.com/rsafonseca), [sahilpatel09](https://github.com/sahilpatel09), [SalehBorhani](https://github.com/SalehBorhani), [SebTardif](https://github.com/SebTardif), [seshachalam-yv](https://github.com/seshachalam-yv), [shashwat010](https://github.com/shashwat010), [shivamgcodes](https://github.com/shivamgcodes), [shuan1026](https://github.com/shuan1026), [silentred](https://github.com/silentred), [sneaky-potato](https://github.com/sneaky-potato), [socketpair](https://github.com/socketpair), [srri](https://github.com/srri), [subrajeet-maharana](https://github.com/subrajeet-maharana), [sxllwx](https://github.com/sxllwx), [tchap](https://github.com/tchap), [tsujiri](https://github.com/tsujiri), [tzfun](https://github.com/tzfun), [upamanyus](https://github.com/upamanyus), [uzairhameed](https://github.com/uzairhameed), [varunu28](https://github.com/varunu28), [vihasmakwana](https://github.com/vihasmakwana), [wendy-ha18](https://github.com/wendy-ha18), [xiaoxiangirl](https://github.com/xiaoxiangirl), [xigang](https://github.com/xigang), [xUser5000](https://github.com/xUser5000), [yagikota](https://github.com/yagikota), [yajianggroup](https://github.com/yajianggroup), [yedou37](https://github.com/yedou37), [Zanda256](https://github.com/Zanda256), [zechariahkasina](https://github.com/zechariahkasina), [zhijun42](https://github.com/zhijun42), [zhoujiaweii](https://github.com/zhoujiaweii)

Відгуки можна надсилати через:

- [Тікети GitHub](https://github.com/etcd-io/etcd/issues)
- [канал Slack #sig-etcd](https://kubernetes.slack.com/archives/C3HD8ARJ5) у [Kubernetes Slack](https://www.kubernetes.dev/docs/comms/slack/#joining-slack)
- [список розсилки etcd-dev](https://groups.google.com/g/etcd-dev)

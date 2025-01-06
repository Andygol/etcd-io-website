---
title: "Довідник API: паралельність"
---

Ця довідка API автоматично згенерована з вказаних файлів `.proto`.

##### service `Lock` (server/etcdserver/api/v3lock/v3lockpb/v3lock.proto)

Сервіс блокування надає клієнтські засоби блокування як інтерфейс gRPC.

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| Lock | LockRequest | LockResponse | Lock отримує розподілений спільний блок на вказаному іменованому блоці. У разі успіху, він поверне унікальний ключ, який існує, поки блок утримується викликачем. Цей ключ можна використовувати разом з транзакціями, щоб безпечно забезпечити оновлення до etcd лише під час утримання блоку. Блок утримується до виклику Unlock на ключі або до закінчення терміну дії оренди, повʼязаної з власником. |
| Unlock | UnlockRequest | UnlockResponse | Unlock приймає ключ, повернутий Lock, і звільняє утримання блоку. Наступний викликач Lock, який чекає на блок, буде розбуджений і отримає право власності на блок. |

##### message `LockRequest` (server/etcdserver/api/v3lock/v3lockpb/v3lock.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name є ідентифікатором для розподіленого спільного блоку, який потрібно отримати. | bytes |
| lease | lease є ID оренди, яка буде прикріплена до права власності на блок. Якщо оренда закінчується або відкликається і наразі утримує блок, блок автоматично звільняється. Виклики до Lock з тією ж орендою будуть розглядатися як одне отримання; блокування двічі з тією ж орендою є бездіяльністю. | int64 |

##### message `LockResponse` (server/etcdserver/api/v3lock/v3lockpb/v3lock.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | etcdserverpb.ResponseHeader |
| key | key є ключем, який існуватиме в etcd протягом часу, коли викликач Lock утримує блок. Користувачі не повинні змінювати цей ключ, інакше блок може проявляти невизначену поведінку. | bytes |

##### message `UnlockRequest` (server/etcdserver/api/v3lock/v3lockpb/v3lock.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| key | key є ключем права власності на блок, наданим Lock. | bytes |

##### message `UnlockResponse` (server/etcdserver/api/v3lock/v3lockpb/v3lock.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | etcdserverpb.ResponseHeader |

##### service `Election` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

Сервіс виборів надає клієнтські засоби виборів як інтерфейс gRPC.

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| Campaign | CampaignRequest | CampaignResponse | Campaign чекає на отримання лідерства у виборах, повертаючи LeaderKey, що представляє лідерство у разі успіху. LeaderKey можна використовувати для видачі нових значень у виборах, транзакційного захисту запитів API на утримання лідерства та відмови від лідерства. |
| Proclaim | ProclaimRequest | ProclaimResponse | Proclaim оновлює опубліковане значення лідера новим значенням. |
| Leader | LeaderRequest | LeaderResponse | Leader повертає поточну прокламацію виборів, якщо така є. |
| Observe | LeaderRequest | LeaderResponse | Observe транслює прокламації виборів у порядку, зробленому обраними лідерами виборів. |
| Resign | ResignRequest | ResignResponse | Resign звільняє лідерство у виборах, щоб інші кандидати могли отримати лідерство у виборах. |

##### message `CampaignRequest` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name є ідентифікатором виборів для кампанії. | bytes |
| lease | lease є ID оренди, прикріпленої до лідерства у виборах. Якщо оренда закінчується або відкликається до відмови від лідерства, то лідерство передається наступному кандидату, якщо такий є. | int64 |
| value | value є початковим проголошеним значенням, встановленим, коли кандидат виграє вибори. | bytes |

##### message `CampaignResponse` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | etcdserverpb.ResponseHeader |
| leader | leader описує ресурси, використані для утримання лідерства у виборах. | LeaderKey |

##### message `LeaderKey` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name є ідентифікатором виборів, що відповідає ключу лідерства. | bytes |
| key | key є непрозорим ключем, що представляє право власності на вибори. Якщо ключ видаляється, то лідерство втрачається. | bytes |
| rev | rev є ревізією створення ключа. Його можна використовувати для перевірки права власності на вибори під час транзакцій, перевіряючи, чи відповідає ревізія створення ключа rev. | int64 |
| lease | lease є ID оренди лідера виборів. | int64 |

##### message `LeaderRequest` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name є ідентифікатором виборів для інформації про лідерство. | bytes |

##### message `LeaderResponse` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | etcdserverpb.ResponseHeader |
| kv | kv є парою ключ-значення, що представляє останнє оновлення лідера. | mvccpb.KeyValue |

##### message `ProclaimRequest` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| leader | leader є утриманням лідерства у виборах. | LeaderKey |
| value | value є оновленням, призначеним для перезапису поточного значення лідера. | bytes |

##### message `ProclaimResponse` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | etcdserverpb.ResponseHeader |

##### message `ResignRequest` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| leader | leader є лідерством, яке потрібно відмовитися шляхом відставки. | LeaderKey |

##### message `ResignResponse` (server/etcdserver/api/v3election/v3electionpb/v3election.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | etcdserverpb.ResponseHeader |

##### message `Event` (api/mvccpb/kv.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| type | type є видом події. Якщо type є PUT, це означає, що нові дані були збережені до ключа. Якщо type є DELETE, це означає, що ключ був видалений. | EventType |
| kv | kv містить KeyValue для події. Подія PUT містить поточну пару kv. Подія PUT з kv.Version=1 означає створення ключа. Подія DELETE/EXPIRE містить видалений ключ з його ревізією модифікації, встановленою на ревізію видалення. | KeyValue |
| prev_kv | prev_kv містить пару ключ-значення до події. | KeyValue |

##### message `KeyValue` (api/mvccpb/kv.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| key | key є ключем у байтах. Порожній ключ не дозволяється. | bytes |
| create_revision | create_revision є ревізією останнього створення цього ключа. | int64 |
| mod_revision | mod_revision є ревізією останньої модифікації цього ключа. | int64 |
| version | version є версією ключа. Видалення скидає версію до нуля, і будь-яка модифікація ключа збільшує його версію. | int64 |
| value | value є значенням, утримуваним ключем, у байтах. | bytes |
| lease | lease є ID оренди, прикріпленої до ключа. Коли прикріплена оренда закінчується, ключ буде видалено. Якщо lease дорівнює 0, то до ключа не прикріплено жодної оренди. | int64 |

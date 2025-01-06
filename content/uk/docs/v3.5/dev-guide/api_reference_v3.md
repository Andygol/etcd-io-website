---
title: Довідник API
---

Ця довідник API автоматично згенерована з вказаних файлів `.proto`.

##### service `Auth` (api/etcdserverpb/rpc.proto)

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| AuthEnable | AuthEnableRequest | AuthEnableResponse | AuthEnable вмикає автентифікацію. |
| AuthDisable | AuthDisableRequest | AuthDisableResponse | AuthDisable вимикає автентифікацію. |
| AuthStatus | AuthStatusRequest | AuthStatusResponse | AuthStatus показує статус автентифікації. |
| Authenticate | AuthenticateRequest | AuthenticateResponse | Authenticate обробляє запит автентифікації. |
| UserAdd | AuthUserAddRequest | AuthUserAddResponse | UserAdd додає нового користувача. Імʼя користувача не може бути порожнім. |
| UserGet | AuthUserGetRequest | AuthUserGetResponse | UserGet отримує детальну інформацію про користувача. |
| UserList | AuthUserListRequest | AuthUserListResponse | UserList отримує список всіх користувачів. |
| UserDelete | AuthUserDeleteRequest | AuthUserDeleteResponse | UserDelete видаляє вказаного користувача. |
| UserChangePassword | AuthUserChangePasswordRequest | AuthUserChangePasswordResponse | UserChangePassword змінює пароль вказаного користувача. |
| UserGrantRole | AuthUserGrantRoleRequest | AuthUserGrantRoleResponse | UserGrant надає роль вказаному користувачу. |
| UserRevokeRole | AuthUserRevokeRoleRequest | AuthUserRevokeRoleResponse | UserRevokeRole відкликає роль вказаного користувача. |
| RoleAdd | AuthRoleAddRequest | AuthRoleAddResponse | RoleAdd додає нову роль. Імʼя ролі не може бути порожнім. |
| RoleGet | AuthRoleGetRequest | AuthRoleGetResponse | RoleGet отримує детальну інформацію про роль. |
| RoleList | AuthRoleListRequest | AuthRoleListResponse | RoleList отримує список всіх ролей. |
| RoleDelete | AuthRoleDeleteRequest | AuthRoleDeleteResponse | RoleDelete видаляє вказану роль. |
| RoleGrantPermission | AuthRoleGrantPermissionRequest | AuthRoleGrantPermissionResponse | RoleGrantPermission надає дозвіл на вказаний ключ або діапазон вказаній ролі. |
| RoleRevokePermission | AuthRoleRevokePermissionRequest | AuthRoleRevokePermissionResponse | RoleRevokePermission відкликає дозвіл на ключ або діапазон вказаної ролі. |

##### service `Cluster` (api/etcdserverpb/rpc.proto)

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| MemberAdd | MemberAddRequest | MemberAddResponse | MemberAdd додає учасника до кластера. |
| MemberRemove | MemberRemoveRequest | MemberRemoveResponse | MemberRemove видаляє існуючого учасника з кластера. |
| MemberUpdate | MemberUpdateRequest | MemberUpdateResponse | MemberUpdate оновлює конфігурацію учасника. |
| MemberList | MemberListRequest | MemberListResponse | MemberList показує всіх учасників кластера. |
| MemberPromote | MemberPromoteRequest | MemberPromoteResponse | MemberPromote підвищує учасника з raft learner (без права голосу) до raft учасника з правом голосу. |

##### service `KV` (api/etcdserverpb/rpc.proto)

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| Range | RangeRequest | RangeResponse | Range отримує ключі в діапазоні з сховища ключ-значення. |
| Put | PutRequest | PutResponse | Put вставляє вказаний ключ у сховище ключ-значення. Запит на вставку збільшує ревізію сховища ключ-значення і генерує одну подію в історії подій. |
| DeleteRange | DeleteRangeRequest | DeleteRangeResponse | DeleteRange видаляє вказаний діапазон зі сховища ключ-значення. Запит на видалення збільшує ревізію сховища ключ-значення і генерує подію видалення в історії подій для кожного видаленого ключа. |
| Txn | TxnRequest | TxnResponse | Txn обробляє кілька запитів в одній транзакції. Запит на транзакцію збільшує ревізію сховища ключ-значення і генерує події з тією ж ревізією для кожного завершеного запиту. Не дозволяється змінювати один і той самий ключ кілька разів в одній транзакції. |
| Compact | CompactionRequest | CompactionResponse | Compact стискає історію подій у сховищі ключ-значення etcd. Сховище ключ-значення повинно періодично стискатися, інакше історія подій буде рости нескінченно. |

##### service `Lease` (api/etcdserverpb/rpc.proto)

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| LeaseGrant | LeaseGrantRequest | LeaseGrantResponse | LeaseGrant створює лізинг, який закінчується, якщо сервер не отримує keepAlive протягом заданого періоду часу. Всі ключі, прикріплені до лізингу, будуть видалені після закінчення терміну дії лізингу. Кожен видалений ключ генерує подію видалення в історії подій. |
| LeaseRevoke | LeaseRevokeRequest | LeaseRevokeResponse | LeaseRevoke відкликає лізинг. Всі ключі, прикріплені до лізингу, будуть видалені. |
| LeaseKeepAlive | LeaseKeepAliveRequest | LeaseKeepAliveResponse | LeaseKeepAlive підтримує лізинг активним, передаючи запити keep alive від клієнта до сервера і відповіді keep alive від сервера до клієнта. |
| LeaseTimeToLive | LeaseTimeToLiveRequest | LeaseTimeToLiveResponse | LeaseTimeToLive отримує інформацію про лізинг. |
| LeaseLeases | LeaseLeasesRequest | LeaseLeasesResponse | LeaseLeases показує всі існуючі лізинги. |

##### service `Maintenance` (api/etcdserverpb/rpc.proto)

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| Alarm | AlarmRequest | AlarmResponse | Alarm активує, деактивує та запитує тривоги щодо справності кластера. |
| Status | StatusRequest | StatusResponse | Status отримує статус учасника. |
| Defragment | DefragmentRequest | DefragmentResponse | Defragment дефрагментує бекенд бази даних учасника для відновлення місця для зберігання. |
| Hash | HashRequest | HashResponse | Hash обчислює хеш всього бекенду ключового простору, включаючи ключі, лізинги та інші сегменти в сховищі. Це призначено ТІЛЬКИ для тестування! Не покладайтеся на це у промисловій експлуатації з поточними транзакціями, оскільки операція Hash не утримує блокування MVCC. Використовуйте API "HashKV" замість цього для перевірки узгодженості сегмента "ключ". |
| HashKV | HashKVRequest | HashKVResponse | HashKV обчислює хеш всіх ключів MVCC до вказаної ревізії. Він ітерує тільки сегмент "ключ" в бекенді сховища. |
| Snapshot | SnapshotRequest | SnapshotResponse | Snapshot надсилає знімок всього бекенду від учасника через потік до клієнта. |
| MoveLeader | MoveLeaderRequest | MoveLeaderResponse | MoveLeader запитує поточного лідера вузла передати своє лідерство іншому вузлу. |
| Downgrade | DowngradeRequest | DowngradeResponse | Downgrade запитує зниження, перевіряє можливість або скасовує зниження версії кластера. Підтримується з версії etcd 3.5. |

##### service `Watch` (api/etcdserverpb/rpc.proto)

| Метод | Тип запиту | Тип відповіді | Опис |
| ----- | ---------- | ------------- | ---- |
| Watch | WatchRequest | WatchResponse | Watch спостерігає за подіями, що відбуваються або вже відбулися. Вхідний і вихідний потоки; вхідний потік створює і скасовує спостерігачів, а вихідний потік надсилає події. Один запит на спостереження може спостерігати за кількома діапазонами ключів, передаючи події для кількох спостережень одночасно. Вся історія подій може бути переглянута, починаючи з останньої ревізії стиснення. |

##### message `AlarmMember` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| memberID | memberID - це ID учасника, повʼязаного з піднятою тривогою. | uint64 |
| alarm | alarm - це тип тривоги, яка була піднята. | AlarmType |

##### message `AlarmRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| action | action - це тип запиту на тривогу. Дія може бути GET для отримання статусів тривог, ACTIVATE для активації тривоги або DEACTIVATE для деактивації піднятої тривоги. | AlarmAction |
| memberID | memberID - це ID учасника, повʼязаного з тривогою. Якщо memberID дорівнює 0, запит на тривогу охоплює всіх учасників. | uint64 |
| alarm | alarm - це тип тривоги, яку слід розглянути для цього запиту. | AlarmType |

##### message `AlarmResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| alarms | alarms - це список тривог, повʼязаних із запитом на тривогу. | (slice of) AlarmMember |

##### message `AuthDisableRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `AuthDisableResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthEnableRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `AuthEnableResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthRoleAddRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name - це імʼя ролі, яку потрібно додати до системи автентифікації. | string |

##### message `AuthRoleAddResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthRoleDeleteRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| role |  | string |

##### message `AuthRoleDeleteResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthRoleGetRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| role |  | string |

##### message `AuthRoleGetResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| perm |  | (slice of) authpb.Permission |

##### message `AuthRoleGrantPermissionRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name - це імʼя ролі, якій буде надано дозвіл. | string |
| perm | perm - це дозвіл, який буде надано ролі. | authpb.Permission |

##### message `AuthRoleGrantPermissionResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthRoleListRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `AuthRoleListResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| roles |  | (slice of) string |

##### message `AuthRoleRevokePermissionRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| role |  | string |
| key |  | bytes |
| range_end |  | bytes |

##### message `AuthRoleRevokePermissionResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthStatusRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `AuthStatusResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| enabled |  | bool |
| authRevision | authRevision - це поточна ревізія сховища автентифікації | uint64 |

##### message `AuthUserAddRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name |  | string |
| password |  | string |
| options |  | authpb.UserAddOptions |
| hashedPassword |  | string |

##### message `AuthUserAddResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthUserChangePasswordRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name - це імʼя користувача, пароль якого змінюється. | string |
| password | password - це новий пароль для користувача. Зверніть увагу, що це поле буде видалено на рівні API. | string |
| hashedPassword | hashedPassword - це новий пароль для користувача. Зверніть увагу, що це поле буде ініціалізовано на рівні API. | string |

##### message `AuthUserChangePasswordResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthUserDeleteRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name | name - це імʼя користувача, якого потрібно видалити. | string |

##### message `AuthUserDeleteResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthUserGetRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name |  | string |

##### message `AuthUserGetResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| roles |  | (slice of) string |

##### message `AuthUserGrantRoleRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| user | user - це імʼя користувача, якому слід надати вказану роль. | string |
| role | role - це імʼя ролі, яку слід надати користувачу. | string |

##### message `AuthUserGrantRoleResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthUserListRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `AuthUserListResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| users |  | (slice of) string |

##### message `AuthUserRevokeRoleRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name |  | string |
| role |  | string |

##### message `AuthUserRevokeRoleResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `AuthenticateRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name |  | string |
| password |  | string |

##### message `AuthenticateResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| token | token - це авторизований токен, який можна використовувати в наступних RPC | string |

##### message `CompactionRequest` (api/etcdserverpb/rpc.proto)

CompactionRequest стискає сховище ключ-значення до вказаної ревізії. Всі замінені ключі з ревізією менше ревізії стиснення будуть видалені.

| Поле | Опис | Тип |
| ---- | ---- | --- |
| revision | revision - це ревізія сховища ключ-значення для операції стиснення. | int64 |
| physical | physical встановлюється так, щоб RPC чекало, поки стиснення фізично не буде застосовано до локальної бази даних, щоб стиснені записи були повністю видалені з бекенду бази даних. | bool |

##### message `CompactionResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `Compare` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| result | result - це логічна операція порівняння для цього порівняння. | CompareResult |
| target | target - це поле ключ-значення для перевірки порівняння. | CompareTarget |
| key | key - це ключ для операції порівняння. | bytes |
| target_union |  | oneof |
| version | version - це версія вказаного ключа | int64 |
| create_revision | create_revision - це ревізія створення вказаного ключа | int64 |
| mod_revision | mod_revision - це остання модифікована ревізія вказаного ключа. | int64 |
| value | value - це значення вказаного ключа, в байтах. | bytes |
| lease | lease - це ID лізингу вказаного ключа. | int64 |
| range_end | range_end порівнює вказаний target з усіма ключами в діапазоні [key, range_end). Дивіться RangeRequest для отримання додаткової інформації про діапазони ключів. | bytes |

##### message `DefragmentRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `DefragmentResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `DeleteRangeRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| key | key - це перший ключ для видалення в діапазоні. | bytes |
| range_end | range_end - це ключ, що слідує за останнім ключем для видалення в діапазоні [key, range_end). Якщо range_end не вказано, діапазон визначається тільки ключем. Якщо range_end на один біт більше, ніж вказаний ключ, то діапазон включає всі ключі з префіксом (вказаний ключ). Якщо range_end дорівнює '\0', діапазон включає всі ключі, більші або рівні ключу. | bytes |
| prev_kv | Якщо prev_kv встановлено, etcd отримує попередні пари ключ-значення перед їх видаленням. Попередні пари ключ-значення будуть повернуті у відповіді на видалення. | bool |

##### message `DeleteRangeResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| deleted | deleted - це кількість ключів, видалених за запитом на видалення діапазону. | int64 |
| prev_kvs | якщо prev_kv встановлено в запиті, попередні пари ключ-значення будуть повернуті. | (slice of) mvccpb.KeyValue |

##### message `DowngradeRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| action | action - це тип запиту на зниження. Дія може бути VALIDATE для перевірки цільової версії, DOWNGRADE для зниження версії кластера або CANCEL для скасування поточного завдання зниження. | DowngradeAction |
| version | version - це цільова версія для зниження. | string |

##### message `DowngradeResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| version | version - це поточна версія кластера. | string |

##### message `HashKVRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| revision | revision - це ревізія сховища ключ-значення для операції хешування. | int64 |

##### message `HashKVResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| hash | hash - це значення хешу, обчислене з ключів MVCC до вказаної ревізії. | uint32 |
| compact_revision | compact_revision - це стиснута ревізія сховища ключ-значення на момент початку хешування. | int64 |

##### message `HashRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `HashResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| hash | hash - це значення хешу, обчислене з бекенду KV. | uint32 |

##### message `LeaseCheckpoint` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID лізингу для контрольної точки. | int64 |
| remaining_TTL | Remaining_TTL - це залишковий час до закінчення терміну дії лізингу. | int64 |

##### message `LeaseCheckpointRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| checkpoints |  | (slice of) LeaseCheckpoint |

##### message `LeaseCheckpointResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `LeaseGrantRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| TTL | TTL - це рекомендований час життя в секундах. Закінчений лізинг поверне -1. | int64 |
| ID | ID - це запитуваний ID для лізингу. Якщо ID встановлено на 0, вибір ID здійснюється орендодавцем. | int64 |

##### message `LeaseGrantResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| ID | ID - це ID лізингу для наданого лізингу. | int64 |
| TTL | TTL - це час життя лізингу, вибраний сервером, в секундах. | int64 |
| error |  | string |

##### message `LeaseKeepAliveRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID лізингу для підтримки активності. | int64 |

##### message `LeaseKeepAliveResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| ID | ID - це ID лізингу з запиту на підтримку активності. | int64 |
| TTL | TTL - це новий час життя для лізингу. | int64 |

##### message `LeaseLeasesRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `LeaseLeasesResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| leases |  | (slice of) LeaseStatus |

##### message `LeaseRevokeRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID лізингу для відкликання. Коли ID відкликано, всі повʼязані ключі будуть видалені. | int64 |

##### message `LeaseRevokeResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `LeaseStatus` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID |  | int64 |

##### message `LeaseTimeToLiveRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID лізингу. | int64 |
| keys | keys - це true для запиту всіх ключів, прикріплених до цього лізингу. | bool |

##### message `LeaseTimeToLiveResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| ID | ID - це ID лізингу з запиту на підтримку активності. | int64 |
| TTL | TTL - це залишковий час життя в секундах для лізингу; лізинг закінчиться через TTL+1 секунд. | int64 |
| grantedTTL | GrantedTTL - це початковий наданий час в секундах при створенні/оновленні лізингу. | int64 |
| keys | Keys - це список ключів, прикріплених до цього лізингу. | (slice of) bytes |

##### message `Member` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID учасника для цього учасника. | uint64 |
| name | name - це імʼя учасника, яке можна прочитати. Якщо учасник не запущений, імʼя буде порожнім рядком. | string |
| peerURLs | peerURLs - це список URL-адрес, які учасник використовує для спілкування з кластером. | (slice of) string |
| clientURLs | clientURLs - це список URL-адрес, які учасник використовує для спілкування з клієнтами. Якщо учасник не запущений, clientURLs будуть порожніми. | (slice of) string |
| isLearner | isLearner вказує, чи є учасник raft learner. | bool |

##### message `MemberAddRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| peerURLs | peerURLs - це список URL-адрес, які доданий учасник буде використовувати для спілкування з кластером. | (slice of) string |
| isLearner | isLearner вказує, чи є доданий учасник raft learner. | bool |

##### message `MemberAddResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| member | member - це інформація про доданого учасника. | Member |
| members | members - це список всіх учасників після додавання нового учасника. | (slice of) Member |

##### message `MemberListRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| linearizable |  | bool |

##### message `MemberListResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| members | members - це список всіх учасників, повʼязаних з кластером. | (slice of) Member |

##### message `MemberPromoteRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID учасника, якого потрібно підвищити. | uint64 |

##### message `MemberPromoteResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| members | members - це список всіх учасників після підвищення учасника. | (slice of) Member |

##### message `MemberRemoveRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID учасника, якого потрібно видалити. | uint64 |

##### message `MemberRemoveResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| members | members - це список всіх учасників після видалення учасника. | (slice of) Member |

##### message `MemberUpdateRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID | ID - це ID учасника, якого потрібно оновити. | uint64 |
| peerURLs | peerURLs - це новий список URL-адрес, які учасник буде використовувати для спілкування з кластером. | (slice of) string |

##### message `MemberUpdateResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| members | members - це список всіх учасників після оновлення учасника. | (slice of) Member |

##### message `MoveLeaderRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| targetID | targetID - це ID вузла для нового лідера. | uint64 |

##### message `MoveLeaderResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |

##### message `PutRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| key | key - це ключ, в байтах, для вставки в сховище ключ-значення. | bytes |
| value | value - це значення, в байтах, для асоціації з ключем у сховищі ключ-значення. | bytes |
| lease | lease - це ID лізингу для асоціації з ключем у сховищі ключ-значення. Значення лізингу 0 вказує на відсутність лізингу. | int64 |
| prev_kv | Якщо prev_kv встановлено, etcd отримує попередню пару ключ-значення перед її зміною. Попередня пара ключ-значення буде повернута у відповіді на вставку. | bool |
| ignore_value | Якщо ignore_value встановлено, etcd оновлює ключ, використовуючи його поточне значення. Повертає помилку, якщо ключ не існує. | bool |
| ignore_lease | Якщо ignore_lease встановлено, etcd оновлює ключ, використовуючи його поточний лізинг. Повертає помилку, якщо ключ не існує. | bool |

##### message `PutResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| prev_kv | якщо prev_kv встановлено в запиті, попередня пара ключ-значення буде повернута. | mvccpb.KeyValue |

##### message `RangeRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| key | key - це перший ключ для діапазону. Якщо range_end не вказано, запит шукає тільки ключ. | bytes |
| range_end | range_end - це верхня межа запитуваного діапазону [key, range_end). Якщо range_end дорівнює '\0', діапазон включає всі ключі >= key. Якщо range_end дорівнює ключу плюс один (наприклад, "aa"+1 == "ab", "a\xff"+1 == "b"), то запит на діапазон отримує всі ключі з префіксом key. Якщо обидва key і range_end дорівнюють '\0', то запит на діапазон повертає всі ключі. | bytes |
| limit | limit - це обмеження на кількість ключів, що повертаються за запитом. Коли limit встановлено на 0, це розглядається як відсутність обмеження. | int64 |
| revision | revision - це точка в часі сховища ключ-значення для використання в діапазоні. Якщо revision менше або дорівнює нулю, діапазон охоплює найновіше сховище ключ-значення. Якщо ревізія була стиснута, ErrCompacted повертається як відповідь. | int64 |
| sort_order | sort_order - це порядок для повернутих відсортованих результатів. | SortOrder |
| sort_target | sort_target - це поле ключ-значення для використання при сортуванні. | SortTarget |
| serializable | serializable встановлює запит на діапазон для використання серіалізованих локальних читань учасника. Запити на діапазон стандартно є лінійними; лінійні запити мають вищу затримку і нижчу пропускну здатність, ніж серіалізовані запити, але відображають поточний консенсус кластера. Для кращої продуктивності, в обмін на можливі застарілі читання, серіалізований запит на діапазон обслуговується локально без необхідності досягати консенсусу з іншими вузлами в кластері. | bool |
| keys_only | keys_only при встановленні повертає тільки ключі, а не значення. | bool |
| count_only | count_only при встановленні повертає тільки кількість ключів у діапазоні. | bool |
| min_mod_revision | min_mod_revision - це нижня межа для повернутих модифікацій ключів; всі ключі з меншими модифікаціями будуть відфільтровані. | int64 |
| max_mod_revision | max_mod_revision - це верхня межа для повернутих модифікацій ключів; всі ключі з більшими модифікаціями будуть відфільтровані. | int64 |
| min_create_revision | min_create_revision - це нижня межа для повернутих ревізій створення ключів; всі ключі з меншими ревізіями створення будуть відфільтровані. | int64 |
| max_create_revision | max_create_revision - це верхня межа для повернутих ревізій створення ключів; всі ключі з більшими ревізіями створення будуть відфільтровані. | int64 |

##### message `RangeResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| kvs | kvs - це список пар ключ-значення, що відповідають запиту на діапазон. kvs порожній, коли запитується кількість. | (slice of) mvccpb.KeyValue |
| more | more вказує, чи є ще ключі для повернення в запитуваному діапазоні. | bool |
| count | count встановлюється на кількість ключів у діапазоні, коли запитується. | int64 |

##### message `RequestOp` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| request | request - це об'єднання типів запитів, прийнятих транзакцією. | oneof |
| request_range |  | RangeRequest |
| request_put |  | PutRequest |
| request_delete_range |  | DeleteRangeRequest |
| request_txn |  | TxnRequest |

##### message `ResponseHeader` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| cluster_id | cluster_id - це ID кластера, який надіслав відповідь. | uint64 |
| member_id | member_id - це ID учасника, який надіслав відповідь. | uint64 |
| revision | revision - це ревізія сховища ключ-значення, коли запит був застосований. Для відповідей на прогрес спостереження, header.revision вказує на прогрес. Всі майбутні події, отримані в цьому потоці, гарантовано матимуть вищий номер ревізії, ніж номер header.revision. | int64 |
| raft_term | raft_term - це термін raft, коли запит був застосований. | uint64 |

##### message `ResponseOp` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| response | response - це об'єднання типів відповідей, повернутих транзакцією. | oneof |
| response_range |  | RangeResponse |
| response_put |  | PutResponse |
| response_delete_range |  | DeleteRangeResponse |
| response_txn |  | TxnResponse |

##### message `SnapshotRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `SnapshotResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header | header містить поточну інформацію про сховище ключ-значення. Перший заголовок у потоці знімків вказує на момент часу знімка. | ResponseHeader |
| remaining_bytes | remaining_bytes - це кількість байтів блобу, які потрібно надіслати після цього повідомлення | uint64 |
| blob | blob містить наступний фрагмент знімка в потоці знімків. | bytes |

##### message `StatusRequest` (api/etcdserverpb/rpc.proto)

Порожнє поле.

##### message `StatusResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| version | version - це версія протоколу кластера, яку використовує відповідний учасник. | string |
| dbSize | dbSize - це розмір фізично виділеної бекенд бази даних, в байтах, відповідного учасника. | int64 |
| leader | leader - це ID учасника, якого відповідний учасник вважає поточним лідером. | uint64 |
| raftIndex | raftIndex - це поточний індекс raft, підтверджений відповідним учасником. | uint64 |
| raftTerm | raftTerm - це поточний термін raft відповідного учасника. | uint64 |
| raftAppliedIndex | raftAppliedIndex - це поточний застосований індекс raft відповідного учасника. | uint64 |
| errors | errors містить інформацію про тривоги/здоровʼя та статус. | (slice of) string |
| dbSizeInUse | dbSizeInUse - це розмір логічно використовуваної бекенд бази даних, в байтах, відповідного учасника. | int64 |
| isLearner | isLearner вказує, чи є учасник raft learner. | bool |

##### message `TxnRequest` (api/etcdserverpb/rpc.proto)

З паперу google paxosdb: Наша реалізація базується на потужному примітиві, який ми називаємо MultiOp. Всі інші операції бази даних, крім ітерації, реалізовані як один виклик до MultiOp. MultiOp застосовується атомарно і складається з трьох компонентів: 1. Список тестів, які називаються guard. Кожен тест у guard перевіряє один запис у базі даних. Він може перевіряти наявність або відсутність значення або порівнювати з заданим значенням. Два різних тести в guard можуть застосовуватися до одного або різних записів у базі даних. Всі тести в guard застосовуються, і MultiOp повертає результати. Якщо всі тести є істинними, MultiOp виконує t op (див. пункт 2 нижче), інакше виконує f op (див. пункт 3 нижче). 2. Список операцій бази даних, які називаються t op. Кожна операція в списку є або вставкою, видаленням, або операцією пошуку і застосовується до одного запису в базі даних. Дві різні операції в списку можуть застосовуватися до одного або різних записів у базі даних. Ці операції виконуються, якщо guard оцінюється як істинний. 3. Список операцій бази даних, які називаються f op. Як і t op, але виконується, якщо guard оцінюється як хибний.

| Поле | Опис | Тип |
| ---- | ---- | --- |
| compare | compare - це список предикатів, що представляють кон'юнкцію термінів. Якщо порівняння вдається, то запити на успіх будуть оброблені в порядку, і відповідь міститиме їх відповідні відповіді в порядку. Якщо порівняння не вдається, то запити на невдачу будуть оброблені в порядку, і відповідь міститиме їх відповідні відповіді в порядку. | (slice of) Compare |
| success | success - це список запитів, які будуть застосовані, коли compare оцінюється як істинний. | (slice of) RequestOp |
| failure | failure - це список запитів, які будуть застосовані, коли compare оцінюється як хибний. | (slice of) RequestOp |

##### message `TxnResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| succeeded | succeeded встановлюється на true, якщо порівняння оцінюється як істинне, або false в іншому випадку. | bool |
| responses | responses - це список відповідей, що відповідають результатам застосування success, якщо succeeded є істинним, або failure, якщо succeeded є хибним. | (slice of) ResponseOp |

##### message `WatchCancelRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| watch_id | watch_id - це ID спостерігача для скасування, щоб більше не передавати події. | int64 |

##### message `WatchCreateRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| key | key - це ключ для реєстрації спостереження. | bytes |
| range_end | range_end - це кінець діапазону [key, range_end) для спостереження. Якщо range_end не вказано, спостерігається тільки ключ. Якщо range_end дорівнює '\0', спостерігаються всі ключі, більші або рівні ключу. Якщо range_end на один біт більше, ніж вказаний ключ, то спостерігаються всі ключі з префіксом (вказаний ключ). | bytes |
| start_revision | start_revision - це необовʼязкова ревізія для спостереження з (включно). Відсутність start_revision означає "зараз". | int64 |
| progress_notify | progress_notify встановлюється так, щоб сервер etcd періодично надсилав WatchResponse без подій новому спостерігачу, якщо немає недавніх подій. Це корисно, коли клієнти бажають відновити відключеного спостерігача, починаючи з недавньої відомої ревізії. Сервер etcd може вирішити, як часто він буде надсилати сповіщення на основі поточного навантаження. | bool |
| filters | filters фільтрують події на стороні сервера перед їх надсиланням спостерігачу. | (slice of) FilterType |
| prev_kv | Якщо prev_kv встановлено, створений спостерігач отримує попереднє ключ-значення перед подією. Якщо попереднє ключ-значення вже стиснуто, нічого не буде повернуто. | bool |
| watch_id | Якщо watch_id надано і він не дорівнює нулю, він буде призначений цьому спостерігачу. Оскільки створення спостерігача в etcd не є синхронною операцією, це можна використовувати для забезпечення правильного порядку при створенні кількох спостерігачів на одному потоці. Створення спостерігача з ID, який вже використовується на потоці, призведе до повернення помилки. | int64 |
| fragment | fragment дозволяє розділяти великі ревізії на кілька відповідей спостереження. | bool |

##### message `WatchProgressRequest` (api/etcdserverpb/rpc.proto)

Запитує, щоб статус прогресу потоку спостереження був надісланий у потоці відповідей спостереження якомога швидше.

Порожнє поле.

##### message `WatchRequest` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| request_union | request_union - це запит на створення нового спостерігача або скасування існуючого спостерігача. | oneof |
| create_request |  | WatchCreateRequest |
| cancel_request |  | WatchCancelRequest |
| progress_request |  | WatchProgressRequest |

##### message `WatchResponse` (api/etcdserverpb/rpc.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| header |  | ResponseHeader |
| watch_id | watch_id - це ID спостерігача, який відповідає відповіді. | int64 |
| created | created встановлюється на true, якщо відповідь стосується запиту на створення спостереження. Клієнт повинен записати watch_id і очікувати отримання подій для створеного спостерігача з того ж потоку. Всі події, надіслані створеному спостерігачу, будуть мати той самий watch_id. | bool |
| canceled | canceled встановлюється на true, якщо відповідь стосується запиту на скасування спостереження. Більше подій не буде надіслано скасованому спостерігачу. | bool |
| compact_revision | compact_revision встановлюється на мінімальний індекс, якщо спостерігач намагається спостерігати на стиснутому індексі. Це відбувається, коли створюється спостерігач на стиснутій ревізії або спостерігач не може наздогнати прогрес сховища ключ-значення. Клієнт повинен розглядати спостерігача як скасованого і не повинен намагатися створити будь-якого спостерігача з тією ж start_revision знову. | int64 |
| cancel_reason | cancel_reason вказує причину скасування спостерігача. | string |
| fragment | fragment є true, якщо велика відповідь спостереження була розділена на кілька відповідей. | bool |
| events |  | (slice of) mvccpb.Event |

##### message `Event` (api/mvccpb/kv.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| type | type - це тип події. Якщо type є PUT, це означає, що нові дані були збережені в ключі. Якщо type є DELETE, це означає, що ключ був видалений. | EventType |
| kv | kv містить KeyValue для події. Подія PUT містить поточну пару ключ-значення. Подія PUT з kv.Version=1 означає створення ключа. Подія DELETE/EXPIRE містить видалений ключ з його модифікаційною ревізією, встановленою на ревізію видалення. | KeyValue |
| prev_kv | prev_kv містить пару ключ-значення до події. | KeyValue |

##### message `KeyValue` (api/mvccpb/kv.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| key | key - це ключ в байтах. Порожній ключ не допускається. | bytes |
| create_revision | create_revision - це ревізія останнього створення цього ключа. | int64 |
| mod_revision | mod_revision - це ревізія останньої модифікації цього ключа. | int64 |
| version | version - це версія ключа. Видалення скидає версію до нуля, а будь-яка модифікація ключа збільшує його версію. | int64 |
| value | value - це значення, яке утримується ключем, в байтах. | bytes |
| lease | lease - це ID лізингу, прикріпленого до ключа. Коли прикріплений лізинг закінчується, ключ буде видалений. Якщо lease дорівнює 0, то до ключа не прикріплено жодного лізингу. | int64 |

##### message `Lease` (server/lease/leasepb/lease.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| ID |  | int64 |
| TTL |  | int64 |
| RemainingTTL |  | int64 |

##### message `LeaseInternalRequest` (server/lease/leasepb/lease.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| LeaseTimeToLiveRequest |  | etcdserverpb.LeaseTimeToLiveRequest |

##### message `LeaseInternalResponse` (server/lease/leasepb/lease.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| LeaseTimeToLiveResponse |  | etcdserverpb.LeaseTimeToLiveResponse |

##### message `Permission` (api/authpb/auth.proto)

Permission - це єдина сутність

| Поле | Опис | Тип |
| ---- | ---- | --- |
| permType |  | Type |
| key |  | bytes |
| range_end |  | bytes |

##### message `Role` (api/authpb/auth.proto)

Role - це єдиний запис у сегменті authRoles

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name |  | bytes |
| keyPermission |  | (slice of) Permission |

##### message `User` (api/authpb/auth.proto)

User - це єдиний запис у сегменті authUsers

| Поле | Опис | Тип |
| ---- | ---- | --- |
| name |  | bytes |
| password |  | bytes |
| roles |  | (slice of) string |
| options |  | UserAddOptions |

##### message `UserAddOptions` (api/authpb/auth.proto)

| Поле | Опис | Тип |
| ---- | ---- | --- |
| no_password |  | bool |

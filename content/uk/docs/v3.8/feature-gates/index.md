---
title: Функціональні можливості
weight: 1175
content_type: concept
card:
  name: reference
  weight: 60
---

<!-- overview -->
Ця сторінка містить огляд різних функціональних можливостей, які адміністратор може вказати для etcd.

Див. [етапи функцій](#feature-stages) для їх пояснення.

<!-- body -->
## Огляд {#overview}

Функціональні можливості — це набір пар ключ=значення, які описують функції etcd. Ви можете увімкнути або вимкнути ці функції за допомогою прапорця `--feature-gates` у командному рядку etcd.

etcd дозволяє увімкнути або вимкнути набір функціональних можливостей. Використовуйте прапорець `-h`, щоб переглянути повний набір функціональних можливостей. Щоб установити функціональні можливості, використовуйте прапорець `--feature-gates` із вказаним списком пар функцій у командному рядку:

```shell
--feature-gates=...,StopGRPCServiceOnDefrag=true
```

Або вкажіть `feature-gates` у файлі конфігурації YAML:

```shell
feature-gates: ...,StopGRPCServiceOnDefrag=true
```

### Зміни в структурі `embed.EtcdServer` {#change-in-embedetcdserver-struct}

У версії 3.6 до `embed.Config` додано поле `ServerFeatureGate`, яке має замінити експериментальні поля, перелічені нижче:

```diff
package embed

type Config struct {
  // Deprecated: Use CompactHashCheck Feature Gate instead. Will be decommissioned in v3.7.
  ExperimentalCompactHashCheckEnabled bool `json:"experimental-compact-hash-check-enabled"`

  // Deprecated: Use InitialCorruptCheck Feature Gate instead. Will be decommissioned in v3.7.
  ExperimentalInitialCorruptCheck bool `json:"experimental-initial-corrupt-check"`

  // Deprecated: Use TxnModeWriteWithSharedBuffer Feature Gate instead. Will be decommissioned in v3.7.
  ExperimentalTxnModeWriteWithSharedBuffer bool `json:"experimental-txn-mode-write-with-shared-buffer"`

  // Deprecated: Use StopGRPCServiceOnDefrag Feature Gate instead. Will be decommissioned in v3.7.
  ExperimentalStopGRPCServiceOnDefrag bool `json:"experimental-stop-grpc-service-on-defrag"`

  // Deprecated: Use LeaseCheckpoint Feature Gate instead. Will be decommissioned in v3.7.
  ExperimentalEnableLeaseCheckpoint bool `json:"experimental-enable-lease-checkpoint"`

  // Deprecated: Use LeaseCheckpointPersist Feature Gate instead. Will be decommissioned in v3.7.
  ExperimentalEnableLeaseCheckpointPersist bool `json:"experimental-enable-lease-checkpoint-persist"`

+ // ServerFeatureGate is a server level feature gate
+ ServerFeatureGate featuregate.FeatureGate
  ...
```

### Функціональні можливості для функцій Alpha або Beta {#feature-gates-for-alpha-or-beta-features}

Наведені нижче таблиці є підсумком функціональних можливостей, які можна встановити для etcd.

| Функція                          | Типово  | Етап  | Деталі                                                                                |
|----------------------------------|---------|-------|---------------------------------------------------------------------------------------|
| CompactHashCheck                 | false   | Alpha |Дозволяє перевіряти цілісність даних перед обслуговуванням будь-якого клієнтського/внутрішнього трафіку.                                                                             |
| InitialCorruptCheck              | false   | Alpha |Дозволяє лідеру періодично перевіряти хеші компакції підлеглих.                                                                          |
| LeaseCheckpoint                  | false   | Alpha |Дозволяє лідеру надсилати регулярні контрольні точки іншим членам кластера, щоб запобігти скиданню залишкового TTL при зміні лідера.                                             |
| LeaseCheckpointPersist           | false   | Alpha |Дозволяє зберігати залишковий TTL, щоб запобігти невизначеному автоматичному поновленню довгострокових ліз.                                                                        |
| SetMemberLocalAddr               | false   | Alpha |Дозволяє використовувати першу вказану адресу, яка не є loopback-адресою з initial-advertise-peer-urls, як локальну адресу для звʼязку з вузлом.      |
| StopGRPCServiceOnDefrag          | false   | Alpha |Дозволяє зупиняти обслуговування клієнтських запитів gRPC-сервісом etcd під час дефрагментації.                                                                     |
| TxnModeWriteWithSharedBuffer     | true    | Beta  |Дозволяє операції запису використовувати спільний буфер у своїх операціях перевірки лише для читання.                                                                              |

## Використання функції {#using-a-feature}

### Етапи функцій {#feature-stages}

Функція може перебувати на етапі *Alpha*, *Beta*, *GA* або *Deprecated*. Етап *Alpha* означає:

* Функціональність типово вимкнена.
* Може містити помилки. Увімкнення функції може виявити помилки.
* Підтримка функції може бути припинена в будь-який момент без попередження.
* API може змінитися несумісним чином у наступному випуску програмного забезпечення без попередження.
* Рекомендується використовувати лише в тестових кластерах з коротким терміном життя, через підвищений ризик помилок та відсутність довгострокової підтримки.

Етап *Beta* означає:

* Функціональність типово увімкнена.
* Функцію ретельно протестовано. Увімкнення функції вважається безпечним.
* Загальна підтримка функції не буде припинена, хоча деталі можуть змінюватися.
* Рекомендується лише для некритичного використання через можливість виявлення нових важковловлюваних помилок при ширшому впровадженні.

{{% alert title="Примітка" color="info" %}}
Будь ласка, спробуйте функції *Beta* та надайте відгук! Після завершення бета-тестування нам може бути складно вносити додаткові зміни.
{{% /alert %}}

Стан *General Availability* (GA), також відомий як *стабільний*, означає:

* Функція завжди увімкнена; ви не можете її вимкнути.
* Відповідна функціональна можливість більше не потрібна.
* Стабільні версії функцій зʼявлятимуться у випущеному програмному забезпеченні протягом багатьох наступних версій.

Стан *Deprecated* означає:

* Функціональна можливість більше не використовується.
* Функція досягла етапу GA або була видалена.

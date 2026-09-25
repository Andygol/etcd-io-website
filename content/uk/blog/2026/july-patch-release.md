---
title: "Липневі патч-випуски Etcd: v3.5.32 та v3.6.13"
author:  "SIG-Etcd Leads"
date: 2026-07-01
draft: false
---

SIG-etcd випустила чергові оновлення у вигляді патчів для гілок версій v3.5 та v3.6. Ці випуски усувають уразливості CVE, пов’язані із залежностями, виправляють помилку автентифікації WebSocket та додають нову опцію, яка допоможе операторам завершити процес виведення з експлуатації сховища v2. Користувачам версій v3.5 та v3.6 слід виконати оновлення під час наступного запланованого періоду технічного обслуговування.

Отримайте оновлення тут:

* [v3.6.13](https://github.com/etcd-io/etcd/releases/tag/v3.6.13)
* [v3.5.32](https://github.com/etcd-io/etcd/releases/tag/v3.5.32)

Офіційні контейнерні образи доступні з [gcr.io](https://gcr.io/etcd-development/etcd).

Як зазначено в [червневому випуску](/blog/2026/june-patch-release/), v3.4 досягла кінця підтримки і більше не отримуватиме жодних патчів. Якщо ви досі працюєте на v3.4, будь ласка, якомога швидше [оновіться до підтримуваної версії](/docs/v3.6/upgrades/upgrade_3_5/).

## Виправлення CVE залежностей {#patching-dependency-cves}

Цей випуск оновлює v3.5 та v3.6 до [golang v1.25.11](https://groups.google.com/g/golang-nuts) та підвищує `go.opentelemetry.io/otel` і `go.opentelemetry.io/otel/sdk` до `v1.43.0` для усунення вразливостей безпеки в go, зокрема [CVE-2026-29181](https://github.com/advisories/GHSA-mh2q-q3fh-2475) та [CVE-2026-39883](https://github.com/advisories/GHSA-hfvc-g4fc-pqhx). v3.6.13 додатково підвищує `golang.org/x/crypto` до `v0.52.0` для вирішення кількох CVE.

Невідомо, скільки з цих вразливостей можна експлуатувати в etcd, але користувачам слід планувати застосування патча якомога швидше незалежно від цього.

Якщо ви знайшли вразливість в etcd, будь ласка, повідомте про неї [через форму на GitHub](https://github.com/etcd-io/etcd/security/advisories/new).

## Виправлення автентифікації websocket із токенами з префіксом bearer {#fixing-websocket-authentication-with-bearer-prefixed-tokens}

Обидва випуски містять виправлення для [автентифікації websocket із токенами автентифікації з префіксом bearer](https://github.com/etcd-io/etcd/pull/21932), яке раніше спричиняло відхилення автентифікованих websocket-запитів, коли токен містив префікс `Bearer`.

## Нова опція `write-only-skip-check` для `--v2-deprecation` {#new-write-only-skip-check-option-for---v2-deprecation}

Обидва випуски додають нове значення [`write-only-skip-check`](https://github.com/etcd-io/etcd/pull/21850) для прапорця `--v2-deprecation`. Воно поводиться як `write-only`, але пропускає перевірку під час запуску, яка інакше перешкоджала б запуску etcd, коли v2-сховище містить власні (не членські) дані.

Ця опція призначена для операторів, які оновлюють etcd з v3.5 до v3.6. Будь ласка, ознайомтеся з посібником [Оновлення etcd з v3.5 до v3.6](/docs/v3.6/upgrades/upgrade_3_6/) перед її використанням; опція є добровільною і має використовуватися на власний ризик оператора. Зверніть увагу, що `write-only-drop-data`, яке стирає будь-які залишкові v2-дані під час запуску, планується стати стандартним в etcd v3.7, тому `write-only-skip-check` дає операторам контрольований шлях для обробки v2-даних у власному темпі до того, як ця зміна відбудеться.

Крім того, v3.5.32 покращує [`etcdutl check v2store`](https://github.com/etcd-io/etcd/pull/21889), щоб перевіряти як v2-знімки, так і записи WAL, тому оператори можуть точно перевірити, які v2-дані залишилися, перш ніж вирішувати, чи використовувати `write-only-skip-check`, чи продовжити повну міграцію.

## Інші покращення у v3.5.32 {#other-improvements-in-v3532}

v3.5.32 переносить зміну [`server: allow non-admin maintenance status`](https://github.com/etcd-io/etcd/pull/21811), яка раніше вийшла у v3.6.12, дозволяючи не-адміністраторам викликати точку доступу обслуговування `Status`.

Цей випуск також містить додаткові виправлення надійності, які можна знайти в повних журналах змін:

* [CHANGELOG-3.6](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.6.md#v3613)
* [CHANGELOG-3.5](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.5.md#v3532)

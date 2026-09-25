---
title: "Патч-випуски Etcd: v3.7.2, v3.6.15 та v3.5.34"
author: "SIG-Etcd Leads"
date: 2026-09-22
draft: false
---

SIG-etcd випустив патч-оновлення для всіх трьох підтримуваних гілок випуску. Ці випуски оновлюють залежності, усувають витік дескриптора файлу під час очищення файлів, виправляють вивід `etcdctl endpoint status` і покращують визначення версії у v3.7. Користувачам v3.5, v3.6 та v3.7 слід оновитися під час наступного запланованого вікна технічного обслуговування після того, як випуски стануть доступними.

Отримайте оновлення тут:

- [v3.7.2](https://github.com/etcd-io/etcd/releases/tag/v3.7.2)
- [v3.6.15](https://github.com/etcd-io/etcd/releases/tag/v3.6.15)
- [v3.5.34](https://github.com/etcd-io/etcd/releases/tag/v3.5.34)

Офіційні контейнерні образи доступні з [gcr.io](https://gcr.io/etcd-development/etcd).

## Оновлення безпеки залежностей {#dependency-security-updates}

v3.6.15 і v3.5.34 оновлюють `github.com/gorilla/websocket` до v1.5.3, щоб [усунути проблему з ненадійною криптографією](https://github.com/advisories/GHSA-w67g-5rqw-f597) (номер CVE ще не присвоєно).

v3.6.15 також оновлює `golang.org/x/text` до v0.39.0, щоб усунути вразливість [CVE-2026-56852](https://pkg.go.dev/vuln/GO-2026-5970).

Усі три випуски компілюють двійкові файли за допомогою [Go 1.26.8](https://go.dev/doc/devel/release).

## Закриття заблокованих файлів після помилок очищення {#close-locked-files-after-purge-failures}

Усі три випуски усувають витік файлового дескриптора й advisory lock у [`purgeFile`](https://github.com/etcd-io/etcd/pull/22452). Якщо видалення заблокованого файлу не вдалося, функція `purgeFile` раніше поверталася, не закриваючи файл. Тепер вона закриває заблокований файл перед поверненням помилки видалення та записує в журнал будь-яку помилку, що виникла під час закриття.

## Виправлення виводу `etcdctl endpoint status` {#correct-etcdctl-endpoint-status-output}

Усі три випуски усувають дубльоване поле `RaftTerm` у виводі `etcdctl endpoint status --write-out=fields`. Тепер команда виводить це поле лише один раз, що полегшує користувачам і автоматизованим системам обробку виводу у форматі полів.

## Більш точне визначення версії у v3.7.2 {#more-accurate-version-detection-in-v372}

v3.7.2 оновлює [`MinimalEtcdVersion`](https://github.com/etcd-io/etcd/pull/22201), щоб читати останній запис зі знімка з журналу записів наперед (WAL). Це запобігає помилці, коли оновлений кластер etcd міг прочитати знімок v2 замість знімка v3.

Повні журнали змін для кожного випуску:

- [CHANGELOG-3.7.2](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.7.md#v372-tbc)
- [CHANGELOG-3.6.15](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.6.md#v3615-tbc)
- [CHANGELOG-3.5.34](https://github.com/etcd-io/etcd/blob/main/CHANGELOG/CHANGELOG-3.5.md#v3534-tbc)

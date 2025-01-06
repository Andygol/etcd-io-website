---
title: Бібліотеки та інструменти
weight: 1300
description: Перелік інструментів та клієнтських бібліотек для etcd
---

Зверніть увагу, що сторонні бібліотеки та інструменти (не розміщені на <https://github.com/etcd-io>), згадані нижче, не тестуються та не підтримуються командою etcd. Перед їх використанням рекомендується ознайомитися з ними та провести власне розслідування.

## Інструменти {#tools}

- [etcdctl](https://github.com/etcd-io/etcd/tree/master/etcdctl) - Клієнт командного рядка для etcd
- [etcd-dump](https://npmjs.org/package/etcd-dump) - Утиліта командного рядка для дампу/відновлення etcd.
- [etcd-fs](https://github.com/xetorthio/etcd-fs) - Файлова система FUSE для etcd
- [etcddir](https://github.com/rekby/etcddir) - Синхронізація etcd та локальної теки в реальному часі. Робота з windows та linux..
- [etcd-browser](https://github.com/henszey/etcd-browser) - Веб-редактор ключів/значень для etcd з використанням AngularJS
- [etcd-lock](https://github.com/datawisesystems/etcd-lock) - Реалізація вибору майстра та розподіленого r/w блокування з використанням etcd - Підтримує v2
- [etcd-console](https://github.com/matishsiao/etcd-console) - Веб-редактор ключів/значень для etcd з використанням PHP
- [etcd-viewer](https://github.com/nikfoundas/etcd-viewer) - Редактор/переглядач сховища ключів-значень etcd, написаний на Java
- [etcdtool](https://github.com/mickep76/etcdtool) - Експорт/Імпорт/Редагування теки etcd у форматах JSON/YAML/TOML та перевірка теки за допомогою JSON-схеми
- [etcdloadtest](https://github.com/sinsharat/etcdloadtest) - Клієнт командного рядка для навантажувального тестування etcd версії 3.0 і вище.
- [lucas](https://github.com/ringtail/lucas) - Вебпереглядач ключів-значень для кластера kubernetes etcd3.0+.
- [etcd-manager](https://etcdmanager.io) - Сучасний, ефективний, багатоплатформний та безкоштовний GUI та клієнтський інструмент для etcd 3.x. Доступний для Windows, Linux та Mac.
- [etcd-backup-restore](https://github.com/gardener/etcd-backup-restore) - Утиліта для періодичного та інкрементного резервного копіювання та відновлення etcd.
- [etcd-druid](https://github.com/gardener/etcd-druid) - Оператор Kubernetes для розгортання кластерів etcd та управління операціями другого дня.
- [etcdadm](https://github.com/kubernetes-sigs/etcdadm) - Інструмент командного рядка для управління кластером etcd.
- [etcd-defrag](https://github.com/ahrtr/etcd-defrag) - Зручний у використанні та розумний інструмент дефрагментації etcd.
- [etcdhelper](https://github.com/tsonglew/intellij-etcdhelper) - Втулок платформи intellij для etcd.

## Бібліотеки {#libraries}

Нижче наведено клієнтські бібліотеки etcd за мовами програмування.

### Go

- [etcd/client/v3](https://github.com/etcd-io/etcd/tree/main/client/v3) - офіційно підтримуваний клієнт Go для v3
- [etcd/client/v2](https://github.com/etcd-io/etcd/tree/release-3.5/client/v2) - офіційно підтримуваний клієнт Go для v2
- [go-etcd](https://github.com/coreos/go-etcd) - застарілий офіційний клієнт. Може бути корисним для старих версій etcd (<2.0.0).
- [encWrapper](https://github.com/lumjjb/etcd/tree/enc_wrapper/clientwrap/encwrapper) - encWrapper - це обгортка шифрування для API ключів клієнта etcd/KV.

### Java

- [coreos/jetcd](https://github.com/etcd-io/jetcd) - Підтримує v3
- [boonproject/etcd](https://github.com/boonproject/boon/blob/master/etcd/README.md) - Підтримує v2, Async/Sync та waits
- [justinsb/jetcd](https://github.com/justinsb/jetcd)
- [diwakergupta/jetcd](https://github.com/diwakergupta/jetcd) - Підтримує v2
- [jurmous/etcd4j](https://github.com/jurmous/etcd4j) - Підтримує v2,Async/Sync, waits та SSL
- [AdoHe/etcd4j](http://github.com/AdoHe/etcd4j) - Підтримує v2 (покращено для реального промислового кластера)
- [cdancy/etcd-rest](https://github.com/cdancy/etcd-rest) - Використовує jclouds для забезпечення повної реалізації API v2.
- [IBM/etcd-java](https://github.com/IBM/etcd-java)

### Scala

- [maciej/etcd-client](https://github.com/maciej/etcd-client) - Підтримує v2. Повністю асинхронний клієнт на основі Akka HTTP
- [eiipii/etcdhttpclient](https://bitbucket.org/eiipii/etcdhttpclient) - Підтримує v2. Асинхронний HTTP клієнт на основі Netty та Scala Futures.
- [mingchuno/etcd4s](https://github.com/mingchuno/etcd4s) - Підтримує v3 з використанням gRPC з опціональною підтримкою Akka Stream.

### Perl

- [hexfusion/perl-net-etcd](https://github.com/hexfusion/perl-net-etcd) - Підтримує v3 grpc gateway HTTP API
- [robn/p5-etcd](https://github.com/robn/p5-etcd) - Підтримує v2

### Python

- [kragniz/python-etcd3](https://github.com/kragniz/python-etcd3) - Клієнт для v3
- [jplana/python-etcd](https://github.com/jplana/python-etcd) - Підтримує v2
- [russellhaering/txetcd](https://github.com/russellhaering/txetcd) - бібліотека Twisted Python
- [cholcombe973/autodock](https://github.com/cholcombe973/autodock) - Інструмент автоматизації розгортання docker
- [lisael/aioetcd](https://github.com/lisael/aioetcd) - (Python 3.4+) Асинхронний клієнт coroutines (Підтримує v2)
- [txaio-etcd](https://github.com/crossbario/txaio-etcd) - Асинхронна бібліотека клієнтів etcd v3 тільки для Twisted (сьогодні) та asyncio (в майбутньому)
- [aioetcd3](https://github.com/gaopeiliang/aioetcd3) - (Python 3.6+) API etcd v3 для asyncio
- [Revolution1/etcd3-py](https://github.com/Revolution1/etcd3-py) - (python2.7 та python3.5+) Клієнт Python для etcd v3, з використанням gRPC-JSON-Gateway

### Node

- [mixer/etcd3](https://github.com/mixer/etcd3) - Підтримує v3
- [stianeikeland/node-etcd](https://github.com/stianeikeland/node-etcd) - Підтримує v2 (з Coffeescript)
- [lavagetto/nodejs-etcd](https://github.com/lavagetto/nodejs-etcd) - Підтримує v2
- [deedubs/node-etcd-config](https://github.com/deedubs/node-etcd-config) - Підтримує v2

### Ruby

- [iconara/etcd-rb](https://github.com/iconara/etcd-rb)
- [jpfuentes2/etcd-ruby](https://github.com/jpfuentes2/etcd-ruby)
- [ranjib/etcd-ruby](https://github.com/ranjib/etcd-ruby) - Підтримує v2
- [davissp14/etcdv3-ruby](https://github.com/davissp14/etcdv3-ruby) - Підтримує v3

### C

- [apache/celix/etcdlib](https://github.com/apache/celix/tree/master/libs/etcdlib) - Підтримує v2
- [jdarcy/etcd-api](https://github.com/jdarcy/etcd-api) - Підтримує v2
- [shafreeck/cetcd](https://github.com/shafreeck/cetcd) - Підтримує v2

### C++

- [edwardcapriolo/etcdcpp](https://github.com/edwardcapriolo/etcdcpp) - Підтримує v2
- [suryanathan/etcdcpp](https://github.com/suryanathan/etcdcpp) - Підтримує v2 (з очікуваннями)
- [nokia/etcd-cpp-api](https://github.com/nokia/etcd-cpp-api) - Підтримує v2
- [nokia/etcd-cpp-apiv3](https://github.com/nokia/etcd-cpp-apiv3) - Підтримує v3

### Clojure

- [aterreno/etcd-clojure](https://github.com/aterreno/etcd-clojure)
- [dwwoelfel/cetcd](https://github.com/dwwoelfel/cetcd) - Підтримує v2
- [rthomas/clj-etcd](https://github.com/rthomas/clj-etcd) - Підтримує v2

### Erlang

- [marshall-lee/etcd.erl](https://github.com/marshall-lee/etcd.erl) - Підтримує v2
- [zhongwencool/eetcd](https://github.com/zhongwencool/eetcd) - Підтримує v3+ (тільки GRPC)

### Elixir

- [team-telnyx/etcdex](https://github.com/team-telnyx/etcdex) - Підтримує v3+ (тільки GRPC)

### .NET

- [wangjia184/etcdnet](https://github.com/wangjia184/etcdnet) - Підтримує v2
- [drusellers/etcetera](https://github.com/drusellers/etcetera)
- [shubhamranjan/dotnet-etcd](https://github.com/shubhamranjan/dotnet-etcd) - Підтримує v3+ (тільки GRPC)
- [SimplifyNet/Etcd.Microsoft.Extensions.Configuration](https://github.com/SimplifyNet/Etcd.Microsoft.Extensions.Configuration)

### PHP

- [linkorb/etcd-php](https://github.com/linkorb/etcd-php)
- [activecollab/etcd](https://github.com/activecollab/etcd)
- [ouqiang/etcd-php](https://github.com/ouqiang/etcd-php) - Клієнт для v3 gRPC gateway

### Haskell

- [wereHamster/etcd-hs](https://github.com/wereHamster/etcd-hs)

### R

- [ropensci/etseed](https://github.com/ropensci/etseed)

### Nim

- [etcd_client](https://github.com/FedericoCeratto/nim-etcd-client)

### Tcl

- [efrecon/etcd-tcl](https://github.com/efrecon/etcd-tcl) - Підтримує v2, за винятком wait.

### Rust

- [jimmycuadra/rust-etcd](https://github.com/jimmycuadra/rust-etcd) - Підтримує v2

### Gradle

- [gradle-etcd-rest-plugin](https://github.com/cdancy/gradle-etcd-rest-plugin) - Підтримує v2

### Lua

- [api7/lua-resty-etcd](https://github.com/api7/lua-resty-etcd) - Підтримує v2 та v3 (grpc gateway HTTP API)

## Інструменти розгортання {#deployment-tools}

### Інтеграції Chef {#chef-integrations}

- [coderanger/etcd-chef](https://github.com/coderanger/etcd-chef)

### Рецепти Chef {#chef-cookbooks}

- [spheromak/etcd-cookbook](https://github.com/spheromak/etcd-cookbook)

### Випуски BOSH {#bosh-releases}

- [cloudfoundry-community/etcd-boshrelease](https://github.com/cloudfoundry-community/etcd-boshrelease)
- [cloudfoundry/cf-release](https://github.com/cloudfoundry/cf-release/tree/master/jobs/etcd)

## Проєкти, що використовують etcd {#projects-using-etcd}

- [Користувачі etcd Raft](https://github.com/etcd-io/etcd/blob/master/raft/README.md#notable-users) - проєкти, що використовують реалізацію бібліотеки raft від etcd.
- [apache/celix](https://github.com/apache/celix) - реалізація специфікації OSGi, адаптована для C та C++
- [binocarlos/yoda](https://github.com/binocarlos/yoda) - etcd + ZeroMQ
- [blox/blox](https://github.com/blox/blox) - колекція відкритих проєктів для управління контейнерами та оркестрування з AWS ECS
- [calavera/active-proxy](https://github.com/calavera/active-proxy) - HTTP-проксі, налаштований за допомогою etcd
- [chain/chain](https://github.com/chain/chain) - програмне забезпечення, призначене для роботи та підключення до високомасштабованих блокчейн-мереж з дозволами
- [derekchiang/etcdplus](https://github.com/derekchiang/etcdplus) - Набір розподілених примітивів синхронізації, побудованих на основі etcd
- [go-discover](https://github.com/flynn/go-discover) - виявлення сервісів на Go
- [gleicon/goreman](https://github.com/gleicon/goreman/tree/etcd) - Гілка клону Go Foreman з підтримкою etcd
- [garethr/hiera-etcd](https://github.com/garethr/hiera-etcd) - бекенд Puppet hiera з використанням etcd
- [mattn/etcd-vim](https://github.com/mattn/etcd-vim) - Встановлення та отримання ключів зсередини vim
- [mattn/etcdenv](https://github.com/mattn/etcdenv) - "env" shebang з інтеграцією etcd
- [kelseyhightower/confd](https://github.com/kelseyhightower/confd) - Управління локальними конфігураційними файлами застосунків за допомогою шаблонів та даних з etcd
- [configdb](https://git.autistici.org/ai/configdb/tree/master) - REST-реляційна абстракція поверх довільних бекендів баз даних, призначена для зберігання конфігурацій та інвентаризацій.
- [kubernetes/kubernetes](https://github.com/kubernetes/kubernetes) - Менеджер кластерів контейнерів, представлений Google.
- [mailgun/vulcand](https://github.com/mailgun/vulcand) - HTTP-проксі, що використовує etcd як бекенд конфігурації.
- [duedil-ltd/discodns](https://github.com/duedil-ltd/discodns) - Простий DNS-сервер імен, що використовує etcd як базу даних для імен та записів.
- [skynetservices/skydns](https://github.com/skynetservices/skydns) - Відповідний RFC DNS-сервер
- [xordataexchange/crypt](https://github.com/xordataexchange/crypt) - Безпечне зберігання значень в etcd з використанням шифрування GPG
- [spf13/viper](https://github.com/spf13/viper) - Бібліотека конфігурації Go, читає значення з ENV, pflags, файлів та etcd з опціональним шифруванням
- [lytics/metafora](https://github.com/lytics/metafora) - Бібліотека розподілених завдань Go
- [ryandoyle/nss-etcd](https://github.com/ryandoyle/nss-etcd) - Модуль GNU libc NSS для вирішення імен з etcd.
- [Gru](https://github.com/dnaeon/gru) - Оркестрування, зроблене легким з Go
- [Vitess](http://vitess.io/) - Vitess - це система кластеризації баз даних для горизонтального масштабування MySQL.
- [lclarkmichalek/etcdhcp](https://github.com/lclarkmichalek/etcdhcp) - DHCP-сервер, що використовує etcd для збереження та координації.
- [openstack/networking-vpp](https://github.com/openstack/networking-vpp) - Драйвер мережі, що програмує [FD.io VPP dataplane](https://wiki.fd.io/view/VPP) для забезпечення віртуальних мереж [OpenStack](https://www.openstack.org/)
- [OpenStack](https://github.com/openstack/governance/blob/master/reference/base-services.rst) - Сервіси OpenStack можуть покладатися на etcd як базову службу.
- [CoreDNS](https://github.com/coredns/coredns/tree/master/plugin/etcd) - CoreDNS - це DNS-сервер, що обʼєднує втулки, частини CNCF та Kubernetes
- [Uber M3](https://github.com/m3db/m3) - M3: Відкрита платформа для великих масштабів метрик від Uber для Prometheus
- [Rook](https://github.com/rook/rook) - Оркестрування зберігання для Kubernetes
- [Patroni](https://github.com/zalando/patroni) - Шаблон для високої доступності PostgreSQL з ZooKeeper, etcd або Consul
- [Trillian](https://github.com/google/trillian) - Trillian реалізує дерево Меркле, вміст якого обслуговується з рівня зберігання даних, що дозволяє масштабуватися до надзвичайно великих дерев.
- [Apache APISIX](https://github.com/apache/apisix) - Apache APISIX - це динамічний, реальний час, високопродуктивний API шлюз.
- [purpleidea/mgmt](https://github.com/purpleidea/mgmt) - Наступне покоління розподіленого, подієвого, паралельного управління конфігурацією!
- [Portworx/kvdb](https://docs.portworx.com/concepts/internal-kvdb/) - Внутрішній kvdb для зберігання конфігурації кластера Portworx.
- [Apache Pulsar](https://pulsar.apache.org/) - Apache Pulsar - це відкрита платформа для обміну повідомленнями та потокової передачі даних, створена для хмари.

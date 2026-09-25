---
title: Спільнота
description: Ласкаво просимо на сторінку користувачів та розробників спільноти etcd
spelling: cSpell:ignore grpcio grpcmeetings subreddit youtube
main_channels:
  - title: >
      [<i class="fab fa-google"></i>Google Group][etcd-dev]
    desc: >
      Приєднуйтесь до форуму [etcd-dev][], щоб ставити запитання та отримувати останні новини про etcd.
  - title: >
      [<i class="fab fa-twitter"></i>Twitter][@etcdio]
    desc: >
      Слідкуйте за нами на [@etcdio][], щоб отримувати оголошення, публікації в блогах та інше.
  - title: >
      [<i class="fab fa-github"></i>Github Discussions][GD]
    desc: >
      Ставте запитання та знаходьте відповіді на свої питання про etcd.
community_resources:
  - title: >
      [<i class="fab"></i>Zoom Meeting][online]
    desc: >
      Приєднуйтесь до учасників та підтримувачів [онлайн][online], кожні два тижні.
  - title: >
      [<i class="fas fa-file-alt"></i>Meeting docs][community-meeting-doc]
    desc: >
      Для деталей зустрічей, зверніться до [документів зустрічей спільноти etcd][community-meeting-doc] та [документів зустрічей з тестування на надійність][Robustness Meeting Notes].
  - title: >
      [<i class="fab fa-youtube"></i>YouTube][etcd-youtube]
    desc: >
      Пропустили зустріч? Не проблема. Дивіться [канал etcd][etcd-youtube] для відео зустрічей.
menu:
  main:
---

{{< blocks/cover color="primary" height="sm" >}}
{{< page/header >}}
{{< /blocks/cover >}}

<div class="container l-container--padded">

<div class="row">
{{< page/toc collapsed=true placement="inline" >}}
</div>

<div class="row">
<div class="col-12 col-lg-8">

{{% alert color="success" %}}
  <i class='fas fa-users mr-1'></i> Наша спільнота цінує повагу та
  інклюзивність. Ми дотримуємося нашого [Кодексу поведінки][Code of Conduct] у всіх взаємодіях.

  [Code of Conduct]: https://www.kubernetes.dev/community/code-of-conduct/
{{% /alert %}}

## Приєднуйтесь до спілкування {#join-the-conversation}

Слідкуйте за цими активними каналами для своєчасних оголошень та підписуйтесь, щоб ставити запитання:

{{% cards "main_channels" %}}

Ви також можете спілкуватися з іншими користувачами та учасниками etcd у каналі #sig-etcd в [Kubernetes Slack][].

## Розклад зустрічей {#meeting-schedule}

Підпишіться на календар SIG-etcd, щоб дізнаватися про розклад усіх наших зустрічей. Ідентифікатор календаря: `be23e1c372820ae5667fad118d1a7a9fd80b5ec75573ae656b3a4bd74fd47193@group.calendar.google.com`

Більшість зустрічей мають спільні порядки денні та нотатки в рамках [документу зустрічей SIG-etcd][community-meeting-doc]. Щоб отримати доступ до документа з нотатками, спочатку потрібно підписатися на [etcd-dev][]. **Паролі Zoom знаходяться в документі зустрічей**.

Будь ласка, зверніться до [спільноти Kubernetes][the Kubernetes community] для отримання додаткової інформації про розклад зустрічей спільноти etcd.

### Зустрічі спільноти {#community-meeting}

* *Мета*: загальне обговорення розробки etcd
* *Розклад*: кожні два тижні, четвер, 11:00 [Тихоокеанський час][Pacific Time]
* *Нотатки*: [Нотатки зі зустрічі спільноти][Community Meeting Notes]
* *Zoom*: [SIG-etcd Zoom][online]

Якщо ви новачок у etcd, ця зустріч, ймовірно, є найкращим місцем для початку! Тут також слід обговорювати пропозиції нових функцій або значних змін у проєкті, а також подальші дії щодо рецензування PR.

### Сортування тікетів {#issue-triage}

* *Мета*: очистити список тікетів та PR
* *Розклад*: кожні два тижні, четвер, 11:00 [Тихоокеанський час][Pacific Time]
* *Нотатки*: [Нотатки зі зустрічі з сортування тікетів][Triage Meeting Notes]
* *Zoom*: [SIG-etcd Zoom][online]

Зустрічі з сортування тікетів чергуються з зустрічами спільноти. Вони спрямовані на те, щоб розібратися з нашим списком PR та тікетів. Зустрічі з сортування тікетів відкриті для будь-якого учасника; вам не обовʼязково бути рецензентом або затверджувачем, щоб допомогти! Вони також можуть бути хорошим способом почати робити внесок.

### Тести на надійність {#robustness-tests}

* *Мета*: робоча зустріч для тестування на надійність
* *Розклад*: кожні два тижні, середа, 11:00 [Тихоокеанський час][Pacific Time]
* *Нотатки*: [Нотатки зі зустрічі з тестування на надійність][Robustness Meeting Notes]
* *Zoom*: [SIG-etcd Zoom][online]

Приєднуйтеся до нас кожні два тижні для спільного дослідження коректності etcd під тиском. Наші цілі полягають у демістифікації тестування розподілених систем шляхом обміну знаннями та сприянням культурі надійного тестування в спільноті etcd, а також у розширенні експертизи шляхом наставництва нових рецензентів та затверджувачів для тестів на надійність etcd. Ми запрошуємо членів спільноти пропонувати пункти для зустрічей.

### Зустрічі з документації {#documentation-meetings}

* *Мета*: робота над документацією, очищення PR з документацією, блог
* *Розклад*: кожні два тижні, вівторок, 17:00 [Тихоокеанський час][Pacific Time]
* *Нотатки*: [Нотатки зі зустрічі з документації][Documentation Meeting Notes]
* *Zoom*: [SIG-etcd Zoom][online]

Хочете допомогти покращити та розширити документацію etcd? Маєте функцію, яку потрібно додати до документації? Або, можливо, ви хочете написати чи відредагувати блог-пост. Ми будемо раді вашій допомозі. Це також може бути хорошою першою зустріччю для вас, якщо ви перебуваєте в часових зонах Австралії, Нової Зеландії, Китаю чи Японії.

### Робоча група Operator {#operator-working-group}

* *Мета*: створення нового etcd operator
* *Розклад*: кожні два тижні, вівторок, 11:00 [Тихоокеанський час][Pacific Time]
* *Нотатки*: [Нотатки WG-etcd-operator][operator-wg-doc]
* *Zoom*: [WG-etcd-operator Zoom][operator-zoom]

Приєднуйтеся до [робочої групи etcd operator][etcd operator working group] для обговорень розробки та управління etcd operator. Ці зустрічі проводяться кожні два тижні і відкриті для всіх членів спільноти, які бажають зробити внесок або залишатися в курсі проекту.

{{% cards "community_resources" %}}

## Внесок {#contributing}

Ваші внески до коду та документації etcd вітаються! Якщо ви знайдете проблему або хочете покращення, створіть issue — або ще краще, розгляньте можливість подання pull request.

Для керівництва щодо внесків до etcd, дивіться [Як зробити свій внесок][How to contribute].

</div>

{{< page/toc placement="sidebar" >}}

</div>

{{< page/page-meta-links >}}

</div>

[@etcdio]: https://twitter.com/etcdio
[etcd-dev]: https://groups.google.com/g/etcd-dev
[etcd-youtube]: https://www.youtube.com/channel/UC7tUWR24I5AR9NMsG-NYBlg
[robustness-tests-meeting-recordings]: https://www.youtube.com/playlist?list=PLRGL688DpO9oF-YEEfVXMzaOUzFYK74-I
[How to contribute]: https://github.com/etcd-io/etcd/blob/main/CONTRIBUTING.md
[community-meeting-doc]: https://docs.google.com/document/d/16XEGyPBisZvmmoIHSZzv__LoyOeluC5a4x353CX0SIM
[Community Meeting Notes]: https://docs.google.com/document/d/16XEGyPBisZvmmoIHSZzv__LoyOeluC5a4x353CX0SIM/edit?tab=t.0#heading=h.txs5bihsv4q0
[Triage Meeting Notes]: https://docs.google.com/document/d/16XEGyPBisZvmmoIHSZzv__LoyOeluC5a4x353CX0SIM/edit?tab=t.xjc2zly8zbof#heading=h.eu3cetgrd3ii
[Documentation Meeting Notes]: https://docs.google.com/document/d/16XEGyPBisZvmmoIHSZzv__LoyOeluC5a4x353CX0SIM/edit?tab=t.gksuxl4c139h
[Robustness Meeting Notes]: https://docs.google.com/document/d/16XEGyPBisZvmmoIHSZzv__LoyOeluC5a4x353CX0SIM/edit?tab=t.v6ew634mcun0#heading=h.eu3cetgrd3ii
[online]: https://zoom.us/j/99252206415
[Pacific Time]: https://www.timeanddate.com/time/zones/pt
[GD]: https://github.com/etcd-io/etcd/discussions
[Kubernetes Slack]: https://slack.k8s.io
[operator-wg-doc]: https://docs.google.com/document/d/1ey4zTTRvtCVJJP2vjF95VjG-sAKlNTcqB2HdmC18Lfc/edit?usp=sharing
[operator-zoom]: https://zoom.us/j/93758419981
[etcd operator working group]: https://github.com/kubernetes/community/tree/master/wg-etcd-operator

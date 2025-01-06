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
      Для деталей зустрічей, зверніться до [документів зустрічей спільноти etcd][community-meeting-doc] та [документів зустрічей з тестування на надійність][robustness-tests-meeting-doc].
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

  [Code of Conduct]: https://github.com/cncf/foundation/blob/master/code-of-conduct.md
{{% /alert %}}

## Приєднуйтесь до спілкування {#join-the-conversation}

Слідкуйте за цими активними каналами для своєчасних оголошень та підписуйтесь, щоб ставити запитання:

{{% cards "main_channels" %}}

Ви також можете спілкуватися з іншими користувачами та учасниками etcd у каналі #sig-etcd в [Kubernetes Slack][].

## Розклад зустрічей {#meeting-schedule}

Будь ласка, зверніться до [спільноти Kubernetes](https://github.com/kubernetes/community/blob/master/sig-etcd/README.md#meetings) для останніх оновлень розкладу зустрічей спільноти etcd.

### Зустрічі спільноти та розгляд питань {#community-meetings-issue-triage}

Учасники та підтримувачі etcd зустрічаються [онлайн][online] щотижня, у **четвер
о 11:00** [Тихоокеанський час][], чергуючи між зустрічами спільноти та розглядом питань. Зустрічі з розгляду питань спрямовані на розгляд нашого беклогу PR та питань. Зустрічі з розгляду питань відкриті для будь-якого учасника; вам не потрібно бути рецензентом або затверджувачем, щоб допомогти! Вони також можуть бути хорошим способом почати вносити свій внесок.

Для інформації про зустріч, дату наступної зустрічі, протоколи минулих зустрічей та записи зустрічей, дивіться [документ зустрічей спільноти etcd][community-meeting-doc].

### Робоча група операторів {#operator-working-group}

Приєднуйтесь до [робочої групи операторів etcd](https://github.com/kubernetes/community/tree/master/wg-etcd-operator) для обговорень щодо розробки та управління оператором etcd. Ці зустрічі проводяться кожні два тижні та відкриті для всіх членів спільноти, які бажають внести свій внесок або залишатися в курсі проєкту.

**Розклад зустрічей:**

- **Кожні два тижні** у **вівторок о 11:00** [Тихоокеанський час][].

**Деталі Zoom:**
Для інформації про зустріч, дату наступної зустрічі, протоколи минулих зустрічей та записи зустрічей, дивіться [документ зустрічей робочої групи операторів][operator-wg-doc].

### Тести на надійність {#robustness-tests}

Приєднуйтесь до нас для спільного дослідження коректності etcd під тиском [онлайн][online] **кожні два тижні**, у **середу о 11:00** [Тихоокеанський час][]. Наші цілі полягають у демістифікації тестування розподілених систем шляхом обміну знаннями та сприянням культурі надійного тестування в спільноті etcd, а також у розширенні експертизи шляхом наставництва нових рецензентів та затверджувачів для тестів на надійність etcd. Ми запрошуємо членів спільноти пропонувати пункти для зустрічей.

Для інформації про зустріч, дату наступної зустрічі, протоколи минулих зустрічей та записи зустрічей, дивіться [документ зустрічей з тестування на надійність][robustness-tests-meeting-doc] та [записи зустрічей з тестування на надійність][robustness-tests-meeting-recordings].

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
[robustness-tests-meeting-doc]: https://docs.google.com/document/d/1idZ_7tV6F18v223LyQ0WVUn9gXLSKyeLwYTdAgbjxpw/edit?usp=sharing
[online]: https://zoom.us/my/cncfetcdproject
[Тихоокеанський час]: https://www.timeanddate.com/time/zones/pt
[GD]: https://github.com/etcd-io/etcd/discussions
[Kubernetes Slack]: https://slack.k8s.io
[operator-wg-doc]: https://docs.google.com/document/d/1ey4zTTRvtCVJJP2vjF95VjG-sAKlNTcqB2HdmC18Lfc/edit?usp=sharing

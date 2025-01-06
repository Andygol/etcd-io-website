---
title: Представляємо sig-etcd
author:  "[Han Kang](https://github.com/logicalhan), [Marek Siarkowicz](https://github.com/serathius), Frederico Muñoz"
date: 2023-11-01
draft: false
---

Спеціальні інтерес-групи (SIG) є фундаментальною частиною проєкту Kubernetes, з великою часткою активності спільноти, що відбувається в них. Коли виникає потреба, [можуть бути створені нові SIG](https://github.com/kubernetes/community/blob/master/sig-wg-lifecycle.md), і саме це сталося нещодавно.

[SIG etcd](https://github.com/kubernetes/community/blob/master/sig-etcd/README.md) є найновішим доповненням до списку SIG Kubernetes. У цій статті ми познайомимося з ним ближче, зрозуміємо його походження, обсяг і плани.

## Критична роль etcd {#the-critical-role-of-etcd}

Якщо ми заглянемо всередину панелі управління кластера Kubernetes, ми знайдемо [etcd](https://kubernetes.io/docs/concepts/overview/components/#etcd), послідовне і високо доступне сховище ключ-значення, яке використовується як основне сховище для всіх даних кластера Kubernetes — це опис саме по собі підкреслює критичну роль, яку відіграє etcd, і його важливість в екосистемі Kubernetes.

Ця критична роль робить стан проєкту та спільноти etcd важливим питанням, і [занепокоєння щодо стану проєкту](https://groups.google.com/a/kubernetes.io/g/steering/c/e-O-tVSCJOk/m/N9IkiWLEAgAJ) на початку 2022 року не залишилися непоміченими. Зміни в команді підтримки, серед інших факторів, сприяли ситуації, яку потрібно було вирішити.

## Чому спеціальна інтерес-група {#why-a-special-interest-group}

З урахуванням критичної ролі etcd було запропоновано, що шлях вперед полягатиме у створенні нової спеціальної інтерес-групи. Якщо etcd вже була в самому серці Кубернети, то створення спеціальної БГЗО не лише визнає цю роль, але й зробить etcd повноправним членом спільноти Kubernetes.

Створення SIG etcd створює спеціальний простір для явного визначення контракту між etcd та Kubernetes api machinery і для запобігання на рівні etcd змін, які порушують цей контракт. Крім того, etcd зможе прийняти процеси, які Kubernetes пропонує своїм SIG ([KEPs](https://www.kubernetes.dev/resources/keps/), [PRR](https://github.com/kubernetes/community/blob/master/sig-architecture/production-readiness.md), [phased feature gates](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/), серед інших) для покращення послідовності та надійності кодової бази. Використання цих процесів буде значною перевагою для спільноти etcd.

Як SIG, etcd також зможе залучати підтримку від учасників Kubernetes: активні внески до etcd від підтримувачів Kubernetes зменшать ймовірність порушення змін у Kubernetes завдяки збільшенню кількості потенційних рецензентів та інтеграції з наявною тестовою інфраструктурою. Це не тільки принесе користь Kubernetes, який зможе краще брати участь і формувати напрямок розвитку etcd з огляду на його критичну роль, але й etcd в цілому.

## Про SIG etcd {#about-sig-etcd}

Нещодавно створена SIG вже працює над досягненням своїх цілей, визначених у її [Статуті](https://github.com/kubernetes/community/blob/master/sig-etcd/charter.md) та [Візії](https://github.com/kubernetes/community/blob/master/sig-etcd/vision.md). Мета зрозуміла: зробити etcd надійним, простим і масштабованим сховищем для побудови хмарних розподілених систем і керування хмарною інфраструктурою за допомогою таких оркестраторів, як Kubernetes.

Повноваження SIG etcd не обмежується виключно компонентом Kubernetes, вони також охоплюють etcd як стандартне рішення. Наша мета — зробити etcd найнадійнішим сховищем ключ-значення, яке можна використовувати будь-де, не обмежуючись специфічними обмеженнями Kubernetes і масштабуючись для задоволення вимог багатьох різноманітних випадків використання.

Ми впевнені, що створення SIG etcd є важливою віхою в життєвому циклі проєкту, одночасно покращуючи сам etcd, а також інтеграцію etcd з Kubernetes. Ми запрошуємо всіх, хто цікавиться etcd, [відвідати нашу сторінку](https://github.com/kubernetes/community/blob/master/sig-etcd/README.md), [приєднатися до нашого каналу в Slack](https://kubernetes.slack.com/messages/etcd) і взяти участь у цьому новому етапі життя etcd.

---
title: Автономне тестування надійності etcd
author: "Marek Siarkowicz (Google)"
date: 2025-10-03
draft: false
---

*Це допис із [блогу CNCF](https://www.cncf.io/blog/2025/09/25/autonomous-testing-of-etcds-robustness/), яким ми також ділимося з нашою спільнотою.*

Як критичний компонент багатьох робочих систем, включно з Kubernetes, головним пріоритетом проєкту etcd є надійність. Забезпечення узгодженості та безпеки даних вимагає від учасників нашого проєкту постійного вдосконалення методологій тестування. У цій статті йдеться про те, як ми використовували передове симуляційне тестування для виявлення непомітних помилок, перевірки надійності наших випусків та підвищення нашої впевненості у стабільності etcd. Ми поділимося нашими ключовими висновками та тим, як вони покращили etcd.

## Покращення тестування надійності etcd {#enhancing-etcds-robustness-testing}

Багато критично важливих систем залежать від правильної та узгодженої роботи etcd, зокрема як основного сховища даних для Kubernetes. Після деяких проблем із випуском v3.5 підтримувачі etcd розробили нову [структуру тестування надійності](https://github.com/etcd-io/etcd/issues/14045) для кращого тестування коректності за різних сценаріїв збоїв. Щоб ще більше розширити наші можливості тестування, ми інтегрували детерміновану платформу симуляційного тестування від [Antithesis](https://antithesis.com/) у наш робочий процес.

Платформа працює, запускаючи весь кластер etcd усередині детермінованого гіпервізора. Це спеціалізоване середовище дає тестовому програмному забезпеченню повний контроль над кожним джерелом недетермінованості, таким як поведінка мережі, планування потоків та системні годинники. Це означає, що будь-яку помилку, яку вона виявляє, можна ідеально та надійно відтворити.

У цьому симульованому середовищі методологія тестування відходить від традиційних сценарних тестів. Замість написання тестів імперативно зі строгими твердженнями для одного конкретного результату цей підхід використовує декларативні, засновані на властивостях твердження про поведінку системи. Ці властивості є високорівневими інваріантами системи, які завжди повинні виконуватися. Наприклад, «узгодженість даних ніколи не порушується» або «подія watch ніколи не втрачається».

Потім платформа розглядає ці властивості не як пасивні перевірки, а як цілі для порушення. Вона поєднує автоматизоване дослідження з цілеспрямованим впровадженням збоїв, активно шукаючи точну послідовність подій і збоїв, які спричинять порушення властивості. Цей активний пошук порушень дозволяє платформі виявляти тонкі помилки, що виникають унаслідок складних комбінацій факторів. Antithesis називає цей підхід автономним тестуванням.

Це ґрунтується на існуючих тестах надійності etcd, які також використовують підхід, заснований на властивостях. Однак без детермінованого середовища чи автоматизованого дослідження початкова структура нагадувала кидання дротиків із завʼязаними очима в надії влучити в яблучко. Помилку можна було знайти, але процес значною мірою покладався на випадковість і його було важко відтворити. Детермінована симуляція та активне дослідження Antithesis знімають повʼязку з очей, уможливлюючи систематичний і відтворюваний пошук помилок.

## Як ми тестували {#how-we-tested}

Наші цілі для цих зусиль з тестування полягали в тому, щоб:

1. **Перевірити надійність etcd v3.6.**
2. **Покращити якість програмного забезпечення etcd, знаходячи та виправляючи помилки.**
3. **Розширити нашу поточну структуру тестування автономним тестуванням.**

Ми запускали наші наявні тести надійності на симуляційній платформі Antithesis, тестуючи кластер etcd із 3 вузлів та 1 вузла проти різноманітних збоїв, включно з:

* **Мережеві збої:** затримка, перевантаження та розділення.
* **Збої на рівні контейнера:** призупинення потоків, завершення процесів, коливання тактової частоти та обмеження продуктивності CPU.

Ми тестували старіші версії etcd із відомими помилками для перевірки методології тестування, а також наші стабільні випуски (3.4, 3.5, 3.6) та головну гілку розробки. Загалом ми провели 830 годин тестування, що дорівнювало 4.5 рокам використання.

## Що ми знайшли {#what-we-found}

Результати були вражаючими. Симуляційне тестування не лише знайшло всі відомі помилки, на які ми тестували, але й виявило кілька нових проблем у нашій головній гілці розробки.

Ось деякі з ключових висновків:

* **Було виявлено критичну помилку watch**, яку пропустили наші наявні тести. Ця помилка була присутня у всіх стабільних випусках etcd.
* **Усі відомі помилки було знайдено**, що дає нам впевненість у здатності комбінованого підходу до тестування виявляти регресії.
* **Наше власне тестування було покращено** шляхом виявлення недоліку в нашій моделі перевірки лінеаризації.

### Проблеми в головній гілці розробки {#issues-in-the-main-development-branch}

| Опис                                                                       | Посилання на звіт    | Статус                                                  | Вплив    | Деталі                                                    |
| :------------------------------------------------------------------------- | :------------------- | :------------------------------------------------------ | :--------| :-------------------------------------------------------- |
| [Watch на майбутній ревізії може отримувати старі події][bug-1-issue]      | [Звіт][bug-1-report] | Виправлено у 3.6.2 ([\#20281][bug-1-fix])               | Середній | Нова помилка, виявлена Antithesis                         |
| [Watch на майбутній ревізії може отримувати старі сповіщення][bug-2-issue] | [Звіт][bug-2-report] | Виправлено у 3.6.2 ([\#20221][bug-2-fix])               | Середній | Нова помилка, виявлена і Antithesis, і тестами надійності |
| [Паніка, коли два знімки отримано за короткий період][bug-3-issue]         | [Звіт][bug-3-report] | Відкрито                                                | Низький  | Раніше виявлено тестами надійності                        |
| [Паніка від сторінки db, очікуваної як 5][bug-4-issue]                     | [Звіт][bug-4-report] | Виправлено у 3.6.5 ([\#20553][bug-4-fix])               | Низький  | Нова помилка, виявлена Antithesis                         |
| [Час операції на основі відповіді watch є неправильним][bug-5-issue]       | [Звіт][bug-5-report] | Виправлено тест у головній гілці ([\#19998][bug-5-fix]) | Низький  | Помилка в тестах надійності, виявлена Antithesis          |

[bug-1-issue]: https://github.com/etcd-io/etcd/issues/20221
[bug-1-report]: https://linuxfoundation.antithesis.com/report/LAbnx9WBHxp0BPeEDSFrTxl3/798H3lSB7pQb6x2LYB65zGlNhM_OmxZAza0PfRbjpQo.html?auth=v2.public.eyJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoiNzk4SDNsU0I3cFFiNngyTFlCNjV6R2xOaE1fT214WkF6YTBQZlJianBRby5odG1sIiwicmVwb3J0X2lkIjoiTEFibng5V0JIeHAwQlBlRURTRnJUeGwzIn19LCJuYmYiOiIyMDI1LTA3LTAyVDA2OjI2OjA5Ljc5MjM0NTQ2OVoifaf6ZskL_GQSGDCZ7ESxV5SbygmAq_NiZZ9Oj2wcMnFOZlEjL5QEfgxM1zjSkF20PrjCjrmKzr4U7fJVJOPT3Qo#/run/e3a65c762111a06ab412abbdec1e3a73-32-6/finding/984b7ce364030642155dcd71d492711c9f9f73a9
[bug-1-fix]: https://github.com/etcd-io/etcd/pull/20281
[bug-2-issue]: https://github.com/etcd-io/etcd/issues/20221
[bug-2-report]: https://linuxfoundation.antithesis.com/report/UZjUP_KGxboJepL7k1q_8pa4/ZqL0Vt9a7YESiiBmGecPMkBP8YgM1IwlTZJ4dcYjmZ8.html?auth=v2.public.eyJuYmYiOiIyMDI1LTA2LTI1VDAzOjE4OjIzLjM4MDU2MDQwMFoiLCJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoiWnFMMFZ0OWE3WUVTaWlCbUdlY1BNa0JQOFlnTTFJd2xUWko0ZGNZam1aOC5odG1sIiwicmVwb3J0X2lkIjoiVVpqVVBfS0d4Ym9KZXBMN2sxcV84cGE0In19feIAsYO4-UIigcL4eMu7QUqA6XFbCU3Hnw7BeyZW06o9x11mFqleHbSbRWdIcLdTH2Xzx42DXNB7dBqYq25Ujg4#/run/e35cadd61e2b01c494095b06141fcc8b-32-6/finding/984b7ce364030642155dcd71d492711c9f9f73a9
[bug-2-fix]: https://github.com/etcd-io/etcd/issues/20221
[bug-3-issue]: https://github.com/etcd-io/etcd/issues/18055
[bug-3-report]: https://linuxfoundation.antithesis.com/report/HqTiW-VhiXU25CCPP8vkSUPB/3Q73gnvlcEpEb6XVWcl4H3qTOnXZ7pFAdkpbpHr8mMI.html?auth=v2.public.eyJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoiM1E3M2dudmxjRXBFYjZYVldjbDRIM3FUT25YWjdwRkFka3BicEhyOG1NSS5odG1sIiwicmVwb3J0X2lkIjoiSHFUaVctVmhpWFUyNUNDUFA4dmtTVVBCIn19LCJuYmYiOiIyMDI1LTA2LTA2VDAxOjA5OjE5Ljc1MDg1NDI2NVoifW8RMYqVcS2V3idTzyvalEO2SnPqycds-Cn710lY-wlfqYPe1MAb2U0R2wEKVwPtSsr79WcnR8yYCyZyCQNqhAc#/run/31f74082d85b5ffdaf9f34ed37480bbd-32-6/finding/138fa550c81efa6efc7170191b75c4a22caea51f
[bug-4-report]: https://linuxfoundation.antithesis.com/report/G-9rIjiZJiwodTEN5avQ7wgK/u_uFsWOwZSxS5mOmbEprwMUijNhsWdV6mfde_CT-y4k.html?auth=v2.public.eyJuYmYiOiIyMDI1LTA3LTAzVDA3OjMxOjUyLjQzMzQ2ODk3NFoiLCJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoidV91RnNXT3daU3hTNW1PbWJFcHJ3TVVpak5oc1dkVjZtZmRlX0NULXk0ay5odG1sIiwicmVwb3J0X2lkIjoiRy05cklqaVpKaXdvZFRFTjVhdlE3d2dLIn19fUHU0wnVRoDtfilwOCROUiDTtcOlIZkrVaddCqjorH3utgcIEPIzlsrMAJGXFC6NTZMneLqAWWU_lq-9prD_tQc#/run/9088730ba7972869a3e2b68b66708b55-32-6/finding/b9cbdf1bc8bd74cab1388e30ebdbf0b37c6f1420
[bug-4-issue]: https://github.com/etcd-io/etcd/issues/20271
[bug-4-fix]: https://github.com/etcd-io/etcd/pull/20553
[bug-5-report]: https://linuxfoundation.antithesis.com/report/IVzVnBQKQ0aInboRbRdsVDIE/xlfYJ3eyHooIxRJqHjimRYPrnttrULyr8PqOfRD0pS8.html?auth=v2.public.eyJuYmYiOiIyMDI1LTA1LTIxVDA5OjMzOjUzLjcyODgwNTA4MVoiLCJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoieGxmWUozZXlIb29JeFJKcUhqaW1SWVBybnR0clVMeXI4UHFPZlJEMHBTOC5odG1sIiwicmVwb3J0X2lkIjoiSVZ6Vm5CUUtRMGFJbmJvUmJSZHNWRElFIn19fc8p5s8qWPm5KxSC8oqMFj8HzTze7dxXhyPVt3l-GLwxSHIsuAIk1-2W7tgrh9mNXpZkFRhedvGSYNyhZ272kAo#/run/2e0ec6758e3603c3e4f5fd43dd26ffab-31-8/finding/5a95b2983bca202814eaa6a3fe594910a72cd2c6
[bug-5-issue]: https://github.com/etcd-io/etcd/issues/19998
[bug-5-fix]: https://github.com/etcd-io/etcd/issues/19998

### Відомі проблеми {#known-issues}

Antithesis також успішно знайшла та відтворила ці відомі проблеми в старіших випусках — «[Brown M&M](https://www.safetydimensions.com.au/van-halen/)s», встановлені підтримувачами etcd.

| Опис                                                                          | Посилання на звіт      |
| :---------------------------------------------------------------------------- | :--------------------- |
| [Watch втрачає подію під час ущільнення при видаленні][known-1-issue]         | [Звіт][known-1-report] |
| [Зменшення ревізії, спричинене збоєм під час ущільнення][known-2-issue]       | [Звіт][known-2-report] |
| [Сповіщення про прогрес Watch не синхронізоване з потоком][known-3-issue]     | [Звіт][known-3-report] |
| [Неузгоджена ревізія, спричинена збоєм під час дефрагментації][known-4-issue] | [Звіт][known-4-report] |
| [Помилка runlock у Watchable][known-5-issue]                                  | [Звіт][known-5-report] |

[known-1-issue]: https://github.com/etcd-io/etcd/issues/18089
[known-1-report]: https://linuxfoundation.antithesis.com/report/eYAhUOXW751VmJwPvGPa6R52/SFgfiy4PFXUGW5JkKt-uOnLFUVk9ZDIxFNQDRIS-eLE.html?auth=v2.public.eyJuYmYiOiIyMDI1LTA2LTAyVDIxOjI5OjMwLjAxNjk5OTQ5NloiLCJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoiU0ZnZml5NFBGWFVHVzVKa0t0LXVPbkxGVVZrOVpESXhGTlFEUklTLWVMRS5odG1sIiwicmVwb3J0X2lkIjoiZVlBaFVPWFc3NTFWbUp3UHZHUGE2UjUyIn19feadk3puhf0lkOv5k8GN_uQ74jb64WhykomO8nUZVBbUqRC-dLOnb7ENYLEjLW_rConu9ADWMFK_WVX7_zpX-wE#/run/6f713ca33a385cfa6d1987f125cbd951-31-8/finding/984b7ce364030642155dcd71d492711c9f9f73a9
[known-2-issue]: https://github.com/etcd-io/etcd/issues/17780
[known-2-report]: https://linuxfoundation.antithesis.com/report/aRbi2JR9dqoXK2xvN-DfZi9S/r4GRi-BLXj6-kpaqz5fo8j8W-qUV1diKw6_x8vonLNk.html?auth=v2.public.eyJuYmYiOiIyMDI1LTA2LTAyVDIxOjMyOjQ5LjU1OTQ3Nzg2MFoiLCJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoicjRHUmktQkxYajYta3BhcXo1Zm84ajhXLXFVVjFkaUt3Nl94OHZvbkxOay5odG1sIiwicmVwb3J0X2lkIjoiYVJiaTJKUjlkcW9YSzJ4dk4tRGZaaTlTIn19fR5EmgiseZ02ngQHRC5uYXTIekPT7Z9Ta903abbN1xq-t2XYheG4YSlJFDdRIfyMpKKclB_uZGQOPd2kXKeutwM#/run/6a08b1cd0efe3d19b9bd89c6815e84e4-31-8/finding/5a95b2983bca202814eaa6a3fe594910a72cd2c6
[known-3-issue]: https://github.com/etcd-io/etcd/issues/15220
[known-3-report]: https://linuxfoundation.antithesis.com/report/ymTYOGwzB-UwlmrjT8VrC_Kn/Y-D2b7S_BKdIl67UqZtXafn0xPhbRulSZVQvPqsBZak.html?auth=v2.public.eyJuYmYiOiIyMDI1LTA1LTI5VDEyOjAyOjIzLjUzMTc3MTg2MloiLCJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoiWS1EMmI3U19CS2RJbDY3VXFadFhhZm4weFBoYlJ1bFNaVlF2UHFzQlphay5odG1sIiwicmVwb3J0X2lkIjoieW1UWU9Hd3pCLVV3bG1yalQ4VnJDX0tuIn19fRmIEwPnKRaq1qnN9tKlGw0m--zs7uFUMMi3AaZM_Kz6Uy0IzsO-af3D1DDBFzSyclF13rqyjI-3ki2d9ufDNQk#/run/fa475411ad6b37641065963bc37b5dd4-31-8/finding/984b7ce364030642155dcd71d492711c9f9f73a9
[known-4-issue]: https://github.com/etcd-io/etcd/pull/14685
[known-4-report]: https://linuxfoundation.antithesis.com/report/kuUVd-WEW4jkcEp7Uzsh-649/doX_RaZAkZxIOxxBn51bdhfjFzrV5ipnJYAQUAT2454.html?auth=v2.public.eyJuYmYiOiIyMDI1LTA2LTE5VDAxOjM3OjAxLjYyODQzMjM4OVoiLCJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoiZG9YX1JhWkFrWnhJT3h4Qm41MWJkaGZqRnpyVjVpcG5KWUFRVUFUMjQ1NC5odG1sIiwicmVwb3J0X2lkIjoia3VVVmQtV0VXNGprY0VwN1V6c2gtNjQ5In19fW1TcMbQfba6iffW4KX_yGOjmwg2qHbRsqzxhJOh8ywc6fxgJa8Lemw1ShkuhQs3caqWHlEEojyAEMVjlLPK4Ac#/run/8b12e3b98a5b206e30c5d0067746083e-32-6/finding/5a95b2983bca202814eaa6a3fe594910a72cd2c6
[known-5-issue]: https://github.com/etcd-io/etcd/pull/13505
[known-5-report]: https://linuxfoundation.antithesis.com/report/6zkMqkEjjJuinArwLZTkJehM/7h42qHA5soBgxYPsMiDa4dSbA-O_g2SL9vkJqxvGON8.html?auth=v2.public.eyJzY29wZSI6eyJSZXBvcnRTY29wZVYxIjp7ImFzc2V0IjoiN2g0MnFIQTVzb0JneFlQc01pRGE0ZFNiQS1PX2cyU0w5dmtKcXh2R09OOC5odG1sIiwicmVwb3J0X2lkIjoiNnprTXFrRWpqSnVpbkFyd0xaVGtKZWhNIn19LCJuYmYiOiIyMDI1LTA1LTMwVDAwOjUyOjM2LjUwMjc3MTMyNFoifSYHsw1ZLCSfxME2keN58uGgi2yHTLvlg5_mFLkmePovjDjan-8SH72WdrmeWc4OMoRR-F3Pmi9UkU546_rtkgI#/run/8b82751143d9baaf98535089301d7af4-31-8/finding/984b7ce364030642155dcd71d492711c9f9f73a9

## Висновок {#conclusion}

Інтеграція цього передового симуляційного тестування в наш робочий процес розробки виявилася успішною. Вона дозволила нам знаходити та виправляти критичні помилки, покращувати нашу наявну структуру тестування та підвищувати нашу впевненість у надійності etcd. Ми продовжуватимемо використовувати цю технологію, щоб гарантувати, що etcd залишається стабільним і надійним розподіленим сховищем ключ-значення для спільноти.

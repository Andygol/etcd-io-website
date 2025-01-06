---
title: Файли постійного сховища etcd
linktitle: Постійне сховище
weight: 2650
description: Довідка про формат постійного сховища та файли
---

Цей документ пояснює формат постійного сховища etcd: іменування, вміст та інструменти, які дозволяють розробникам перевіряти їх. У майбутньому документ буде доповнюватися змінами в моделі зберігання даних. Цей документ призначений для розробників etcd, щоб допомогти їм у відновленні даних.

## Передумови {#prerequisites}

Наступні статті надають корисну інформацію для цього документа:

* Огляд моделі даних etcd: [/docs/v3.5/learning/data_model](/docs/v3.5/learning/data_model/)
* Огляд Raft: <https://raft.github.io/raft.pdf> (особливо розділ "5.3 Реплікація журналу").


## Огляд {#overview}

### Довготривалі файли {#long-leaving-files}

<table>
  <tr>
    <th>Назва файлу</th>
    <th>Основна мета</th>
  </tr>

  <tr>
    <td><pre>./member/snap/db</pre></td>
    <td><strong>bbolt <a href="https://uk.wikipedia.org/wiki/B%2B_дерево">b+tree</a></strong>, що зберігає всі застосовані дані, інформацію про авторизацію членства та метадані. Воно знає, який останній застосований індекс журналу WAL (<a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/etcdserver/cindex/cindex.go#L92">"consistent_index"</a>).
    </td>
  </tr>

  <tr>
    <td><pre>./member/snap/0000000000000002-0000000000049425.snap
./member/snap/0000000000000002-0000000000061ace.snap</pre>
    </td>
    <td>
      <p>
        Періодичні <strong>знімки старого сховища v2</strong>, що містять:
        <ul>
          <li>основну інформацію про членство</li>
          <li>версію etcd</li>
        </ul>
      </p>
      <p>З версії etcd v3, вміст дублюється у файлах /snap/db.</p>
      <p>Періодично (<a href="https://github.com/etcd-io/etcd/blob/a4570a60e771402360755beb7d662bdbca1f87f2/server/etcdserver/server.go#L87">30с</a>) ці файли <a href="https://github.com/etcd-io/etcd/blob/aa97484166d2b3fb6afeb4390344e68b02afb566/server/etcdserver/server.go#L597">видаляються</a>, і зберігаються останні <code>--max-snapshots=5</code>.</p>
   </td>
  </tr>

  <tr>
    <td><pre>/member/snap/000000000007a178.snap.db</pre></td>
    <td>
      <p>Повний <strong>знімок bbolt</strong>, завантажений з лідера etcd, якщо репліка відставала занадто сильно.</p>
      <p>Має такий самий тип вмісту, як і файл (<code>./member/snap/db</code>).</p>
      <p>
        Файл використовується у 2 сценаріях:
        <ul>
          <li>У відповідь на запит лідера для відновлення зі знімка.</li>
          <li><a href="https://github.com/etcd-io/etcd/blob/a4570a60e771402360755beb7d662bdbca1f87f2/server/etcdserver/server.go#L444">Під час запуску сервера</a>, коли останній знімок (.snap.db файл) виявляється новішим за consistent_index у поточному файлі <code>snap.db</code>.</li>
        </ul>
        Примітка: Періодичні знімки, створені на кожній репліці, випускаються лише у формі *.snap файлу (не snap.db файлу). Тому немає гарантії, що найновіший знімок (у журналі WAL) має *.snap.db файл. Але в такому випадку бекенд (snap/db) очікується новішим за знімок.
      </p>
      <p>
        Файл <a href="https://github.com/etcd-io/etcd/blob/a4570a60e771402360755beb7d662bdbca1f87f2/server/etcdserver/server.go#L1236">не видаляється після завершення відновлення</a> (тому весь вміст переноситься у файл ./member/snap/db). Періодично (<a href="https://github.com/etcd-io/etcd/blob/a4570a60e771402360755beb7d662bdbca1f87f2/server/etcdserver/server.go#L87">30с</a>) файли <a href="https://github.com/etcd-io/etcd/blob/a4570a60e771402360755beb7d662bdbca1f87f2/server/etcdserver/server.go#L774">видаляються</a>.
Тут також зберігаються <code>--max-snapshots=5</code>. Оскільки ці файли можуть бути O(GBs), це може створити ризик вичерпання дискового простору.
    </td>
  </tr>

  <tr>
    <td><pre>./member/wal/000000000000000f-00000000000b38c7.wal
./member/wal/000000000000000e-00000000000a7fe3.wal
./member/wal/000000000000000d-000000000009c70c.wal</pre>
    </td>
    <td>
      <p><strong>Журнали попереднього запису Raft</strong>, що містять останні транзакції, прийняті Raft, періодичні знімки або записи CRC.</p>
      <p>Зберігаються останні <code>--max-wals=5</code> файлів. Кожен з цих файлів має розмір <code>~64*10^6</code> байтів. Файл розрізається, коли перевищує цей жорстко закодований розмір, тому файли можуть трохи перевищувати цей розмір (тому попередньо виділений <code>0.tmp</code> не забезпечує повного захисту від перевищення дискового простору).</p>
      <p>Якщо знімки робляться занадто рідко, може бути більше ніж <code>--max-wals=5</code>, оскільки файлові системи захищають файли, запобігаючи їх передчасному видаленню.</p>
    </td>
  </tr>

  <tr>
    <td><pre>./member/wal/0.tmp (або .../1.tmp)</pre></td>
    <td>
      <strong>Попередньо виділений простір</strong> для наступного файлу журналу попереднього запису.
      Використовується для запобігання зупинці Raft через відсутність можливості журналів WAL без можливості підняти тривогу.
    </td>
  </tr>
</table>


### Тимчасові файли {#temporary-files}

Під час внутрішньої обробки etcd можливо, що можуть зʼявитися кілька короткотривалих файлів:

<table>
  <tr>
    <th>Файл</th>
    <th>Основна мета</th>
  </tr>
  <tr>
    <td><pre>./member/snap/0000000000000002-000000000007a178.snap.broken</pre></td>
    <td>
      <p>Файли знімків перейменовуються на ‘broken’, коли їх не можна <a href="https://github.com/etcd-io/etcd/blob/aa97484166d2b3fb6afeb4390344e68b02afb566/server/etcdserver/api/snap/snapshotter.go#L144">завантажити</a>.<p>
      <!-- TODO: etcdserver/api/snap/snapshotter.go:148 -->
      <p>Спроба завантажити найновіший файл відбувається під час <a href="https://github.com/etcd-io/etcd/blob/aa97484166d2b3fb6afeb4390344e68b02afb566/server/etcdserver/server.go#L302">запуску</a> etcd.</p>
      <!-- TODO: etcdserver/server.go:428 -->
      <p>Або під час команд резервного копіювання/міграції etcdctl.</p>
   </td>
  </tr>
  <tr>
    <td><pre>./member/snap/tmp071677638 (випадковий суфікс)</pre></td>
    <td>
      <p><a href="https://github.com/etcd-io/etcd/blob/ae9734ed278b7a1a7dfc82e800471ebbf9fce56f/etcdserver/api/snap/db.go#L39">Тимчасовий</a> (bbolt) файл, створений на репліках у відповідь на запит msgSnap лідера, тобто на вимогу лідера відновити сховище зі знімка.</p>
      <p>
        Після успішного (повного) отримання вмісту файл <a href="https://github.com/etcd-io/etcd/blob/ae9734ed278b7a1a7dfc82e800471ebbf9fce56f/etcdserver/api/snap/db.go#L55">перейменовується</a> на: <code>/member/snap/[SNAPSHOT-INDEX].snap.db</code>. У разі зупинки сервера / його завершення під час завантаження файлів, файли залишаються на диску і ніколи не видаляються автоматично. Вони можуть бути значними за розміром (GBs).
      </p>
      <p>Див. <a href="https://github.com/etcd-io/etcd/issues/12837">etcd/issues/12837</a>. <strong>Виправлено у версії etcd 3.5.</strong></p>
    </td>
  </tr>
  <tr>
    <td><pre>/member/snap/db.tmp.071677638 (випадковий суфікс)</pre></td>
    <td>
      <p>Тимчасовий файл, що містить копію вмісту бекенду (/member/snap/db), під час <a href="https://github.com/etcd-io/etcd/blob/a4570a60e771402360755beb7d662bdbca1f87f2/server/mvcc/backend/backend.go#L373">процесу дефрагментації</a>. Після успішного процесу файл перейменовується на /member/snap/db, замінюючи оригінальний бекенд.</p>
      <p>Під час запуску сервера etcd ці файли <a href="https://github.com/etcd-io/etcd/blob/a4570a60e771402360755beb7d662bdbca1f87f2/server/etcdserver/api/snap/snapshotter.go#L217">видаляються</a>.</p>
    </td>
  </tr>
</table>

## bbolt b+tree: **member/snap/db**

Цей файл містить основний вміст etcd, застосований до певної точки журналу Raft (див. [consistent_index](https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/etcdserver/cindex/cindex.go#L92)).


### Фізична організація {#physical-organization}

Краще сховище bolt фізично організоване як [b+tree](https://uk.wikipedia.org/wiki/B%2B_дерево). Фізичні сторінки b-tree ніколи не змінюються на місці[^1]. Натомість вміст копіюється на нову сторінку (відновлену зі списку вільних сторінок), а стара сторінка додається до списку вільних сторінок, як тільки немає відкритої транзакції, яка може отримати до неї доступ. Завдяки цьому процесу відкрита транзакція RO бачить послідовний історичний стан сховища. Транзакція RW є ексклюзивною та блокує всі інші транзакції RW.  \
Великі значення зберігаються на кількох безперервних сторінках. Процес відновлення сторінок у поєднанні з необхідністю виділення суміжних областей сторінок різного розміру може призвести до зростання фрагментації сховища bbolt.

Файл bbolt ніколи не зменшується самостійно. Лише під час процесу дефрагментації файл можна переписати на новий, який має деякий буфер вільних сторінок у кінці та має скорочений розмір.


### Логічна організація {#logical-organization}

Сховище bbolt поділено на кошики. У кожному кошику зберігаються ключі (пари byte[]->value byte[]), у лексикографічному порядку. Нижче наведено список кошиків, які використовуються etcd (станом на версію 3.5) та ключів, що використовуються.

<table>
  <tr>
    <th>Кошик</th>
    <th>Ключ</th>
    <th>Зразкове значення</th>
    <th>Опис</th>
  </tr>
  <tr>
    <td>alarm</td>
    <td><code><a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/api/etcdserverpb/rpc.proto#L976">rpcpb.Alarm</a>:
    {MemberID, Alarm: NONE|NOSPACE|CORRUPT}</code>
    </td>
    <td><code>nil</code></td>
    <td>Вказує на проблеми, діагностовані в одному з членів.</td>
  </tr>
  <tr>
    <td>auth</td>
    <td>"authRevision"</td>
    <td><code>""</code> (порожній) або <code>BigEndian.PutUint64</code></td>
    <td>
      <p>Будь-яка зміна ролей або користувачів збільшує це поле під час коміту транзакції.</p>
      <p>
        Значення використовується лише для <a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/etcdserver/v3_server.go#L461">оптимістичного блокування під час</a> процесу авторизації.
      </p>
    </td>
  </tr>
  <tr>
    <td>authRoles</td>
    <td>[roleName] як string</td>
    <td><code><a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/api/authpb/auth.proto#L38">authpb.Role</a></code> серіалізований</td>
    <td></td>
  </tr>
  <tr>
    <td>authUsers</td>
    <td>[userName] як string</td>
    <td><code><a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/api/authpb/auth.proto#L17">authpb.User</a></code> серіалізований</td>
    <td></td>
  </tr>
  <tr>
    <td rowspan="2" >cluster</td>
    <td>"clusterVersion"</td>
    <td><code>"3.5.0"</code> (string)</td>
    <td><a href="https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/etcdserver/server.go#L2314">мінорна</a> версія узгодженої версії спільного сховища.</td>
  </tr>
  <tr>
    <td>"downgrade"</td>
    <td>JSON: <pre>{
  "target-version": "3.4.0"
  "enabled": true/false
}</pre>
    </td>
    <td>
      <p>Зберігає намір, налаштований останнім запитом <code>Downgrade RPC</code>.</p>
      <p>З версії v3.5</p>
    </td>
  </tr>
  <tr>
    <td><strong>key</strong></td>
    <td>
      <p>[revisionId] закодований за допомогою <a href="https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/mvcc/revision.go#L56">bytesToRev</a>{main,sub}</p>
      <p>Ключ-значення видалення серіалізуються з 't' в кінці (як "Tombstone")</p>
    </td>
    <td><a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/api/mvccpb/kv.proto#L12"><code>mvccpb.KeyValue</code></a> серіалізований proto (<code>key, create_rev, mod_rev, version, value, lease id</code>)</td>
    <td></td>
  </tr>
  <tr>
    <td>lease</td>
    <td></td>
    <td><a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/lease/leasepb/lease.proto#L13"><code>leasepb.Lease</code></a> серіалізований proto (ID, TTL, RemainingTTL)</td>
    <td>
      <p>Примітка: LeaseCheckpoint розширює лише RemainingTTL. Просто TTL з оригінального Grant.</p>
      <p style="text-decoration:underline;">Примітка2: Ми зберігаємо TTL у секундах (з невизначеного 'зараз'). Сервер, що зазнає краху, не звільняє оренди!!!</p>
    </td>
  </tr>
  <tr>
    <td>members</td>
    <td>[memberId] у hex string: <code>"8e9e05c52164694d"</code></td>
    <td>
      <em>JSON як рядок серіалізованої <a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/etcdserver/api/membership/member.go#L43">Member</a> структури:</em>
      <pre>{
  "id":10276657743932975437,
  "peerURLs":[
  "<a href="http://localhost:2380">http://localhost:2380</a>"],
  "name":"default",
  "clientURLs": ["http://localhost:2379"]
}</pre>
    </td>
    <td>Узгоджена інформація про членство в кластері.</td>
  </tr>
  <tr>
    <td>members_removed</td>
    <td>[memberId] у hex string: <code>"8e9e05c52164694d"</code></td>
    <td><code>[]byte("removed")</code></td>
    <td>
      <p>Ідентифікатори всіх видалених членів. Використовується для перевірки, що видалений член ніколи не додається знову під тим самим ідентифікатором.</p>
      <p><a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/etcdserver/api/membership/cluster.go#L251">Поле наразі (3.4) читається лише зі сховища V2 і ніколи з V3</a>. Див. <a href="https://github.com/etcd-io/etcd/pull/12820">https://github.com/etcd-io/etcd/pull/12820</a></p>
    </td>
  </tr>
  <tr>
    <td rowspan="3"><strong>meta</strong></td>
    <td><a href="https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/etcdserver/cindex/cindex.go#L92">"consistent_index"</a></td>
    <td>uint64 байти (BigEndian)</td>
    <td>Представляє зміщення останнього застосованого запису журналу WAL до сховища bolt DB.</td>
  </tr>
  <tr>
    <td><a href="https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/mvcc/kvstore.go#L41">"scheduledCompactRev"</a></td>
    <td><a href="https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/mvcc/revision.go#L56">bytesToRev</a>{main,sub} закодований. (16 байтів)</td>
    <td>Використовується для повторної ініціалізації стиснення, якщо сталася аварія після запиту на стиснення.</td>
  </tr>
  <tr>
    <td><a href="https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/mvcc/kvstore.go#L42">"finishedCompactRev"</a></td>
    <td><a href="https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/mvcc/revision.go#L56">bytesToRev</a>{main,sub} закодований. (16 байтів)</td>
    <td>Ревізія, на якій сховище було нещодавно успішно стиснуто (<a href="https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/mvcc/kvstore_compaction.go#L54">https://github.com/etcd-io/etcd/blob/ae7862e8bc8007eb396099db4e0e04ac026c8df5/server/mvcc/kvstore_compaction.go#L54</a>)</td>
  </tr>
  <tr>
    <td></td>
    <td>"confState"</td>
    <td></td>
    <td><a href="https://github.com/etcd-io/etcd/pull/12962">З версії etcd 3.5</a></td>
  </tr>
  <tr>
    <td></td>
    <td>"term"</td>
    <td></td>
    <td><a href="https://github.com/etcd-io/etcd/pull/12964">З версії etcd 3.5</a></td>
  </tr>
  <tr>
    <td></td>
    <td>"storage-version"</td>
    <td></td>
    <td></td>
  </tr>
</table>

### Інструменти {#tools}

#### bbolt

bbolt має командний інструмент, який дозволяє перевіряти вміст файлу.

Приклади використання:

##### Перелік всіх кошиків у вказаному файлі bbolt: {#list-all-buckets-in-given-bbolt-file}

```sh
% go run go.etcd.io/bbolt/cmd/bbolt buckets ./default.etcd/member/snap/db
```

##### Прочитати конкретну пару ключ/значення: {#read-a-particular-keyvalue-pair}

```sh
% go run go.etcd.io/bbolt/cmd/bbolt get ./default.etcd/member/snap/db cluster clusterVersion
```

#### etcd-dump-db

etcd-dump-db можна використовувати для переліку вмісту бекенду v3 etcd (bbolt).

```sh
% go run go.etcd.io/etcd/v3/tools/etcd-dump-db  list-bucket default.etcd
alarm
auth
...
```

Див. більше прикладів у: [https://github.com/etcd-io/etcd/tree/master/tools/etcd-dump-db](https://github.com/etcd-io/etcd/tree/master/tools/etcd-dump-db)

## WAL: Журнал попереднього запису {#wal-writes-ahead-log}

Журнал попереднього запису (Write ahead log) є постійним сховищем Raft, яке використовується для зберігання пропозицій. Спочатку лідер зберігає пропозицію у своєму журналі, а потім (одночасно) реплікує її за допомогою протоколу Raft до послідовників. Кожен послідовник зберігає пропозицію у своєму журналі WAL перед підтвердженням реплікації лідеру.

Журнал WAL, що використовується в etcd, відрізняється від канонічної моделі Raft двома способами:

* Він зберігає не тільки індексовані записи, але й знімки Raft (легкі) та жорсткий стан. Таким чином, весь стан Raft члена можна відновити лише з журналу WAL.
* Він призначений лише для додавання. Записи не перевизначаються на місці, але запис, доданий пізніше у файлі (з тим самим індексом), витісняє попередній.



### Імена файлів {#file-names}

Файли журналу WAL називаються за наступним шаблоном:

```console
"%016x-%016x.wal", seq, index
```

Приклад: `./member/wal/0000000000000010-00000000000bf1e6.wal`

Отже, імена файлів містять шістнадцяткові коди:


* Послідовний номер файлу журналу WAL
* Індекс першого запису або знімка у файлі. Зокрема, перший файл “0000000000000000-0000000000000000.wal” має початковий запис знімка з індексом=0.

### Фізичний вміст {#physical-content}

Файл журналу WAL містить послідовність "[Фреймів](https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/wal/encoder.go#L62)". Кожен фрейм містить:


1. [LittleEndian](https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/wal/encoder.go#L120)[^2] закодований uint64, що містить довжину серіалізованого [walpb.Record](https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/wal/walpb/record.proto#L11) (3).
2. Відступ: Деяка кількість 0 байт, така, що весь кадр має вирівняний (mod 8) розмір
3. Серіалізовані дані [walpb.Record](https://github.com/etcd-io/etcd/blob/a1ff0d5373335665b3e5f4cb22a538ac63757cb6/server/wal/walpb/record.proto#L11):
    1. [type](https://github.com/etcd-io/etcd/blob/aa97484166d2b3fb6afeb4390344e68b02afb566/server/storage/wal/wal.go#L39) — int закодований enum, що керує інтерпретацією поля даних нижче
    2. дані — залежно від типу, зазвичай серіалізований proto
    3. crc — RC-32 контрольна сума всіх полів “data” (без типу) у всіх записах журналу на цій конкретній репліці з моменту створення журналу WAL. Зверніть увагу, що CRC враховує ВСІ записи (навіть якщо вони не були підтверджені Raft).

Файли "розрізаються" (починається новий файл), коли поточний файл перевищує `64*10^6` байтів.

### Логічний вміст {#logical-content}

Файли журналу попереднього запису на логічному рівні містять:

* `Raftpb.Entry: `останні пропозиції, репліковані лідером Raft. Деякі з цих пропозицій вважаються «підтвердженими», а інші можуть бути логічно перезаписані.
* `Raftpb.HardState(term,commit,vote): `періодична (дуже часта) інформація про індекс запису журналу, який є «підтвердженим» (реплікованим більшістю серверів), тому гарантовано не змінюється/перезаписується і може бути застосований до бекендів (v2, v3). Він також містить “term” (індикатор, чи були якісь зміни, пов’язані з виборами) та голос — член, за якого поточна репліка проголосувала у поточному терміні.
* `walpb.Snapshot(term, index): `періодичні знімки стану Raft (без вмісту DB, лише індекс знімка журналу та термін Raft)
    * Вміст сховища V2 зберігається в окремих файлах *.store.
    * Вміст сховища V3 підтримується у файлі bbolt, і він стає неявним знімком, як тільки записи застосовуються там.
* запис контрольної суми crc32 (на початку кожного файлу), використовується для відновлення перевірки CRC для решти файлу.
* `etcdserverpb.Metadata(node_id, cluster_id)` — ідентифікація кластера та репліки, яку представляє журнал.

Кожен файл журналу WAL складається з (у порядку):

1. Фрейм CRC-32 (поточний crc з усіх попередніх файлів, 0 для першого файлу).
2. Фрейм метаданих (ідентифікатори кластера та репліки)
3. Для початкового файлу WAL:

    * Порожній фрейм знімка (Index:0, Term: 0). Мета цього фрейму — підтримувати інваріант, що всі записи «передують» знімку.

   Для не початкового (2-го+) файлу WAL:

    * Фрейм жорсткого стану.

4. Змішування записів, жорсткого стану та знімків

Журнал WAL може містити кілька записів для одного індексу. Така ситуація може статися у випадках, описаних на малюнку 7. у [статті про Raft](http://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14.pdf). Журнал etcd WAL лише доповнюється, тому записи перевизначаються шляхом додавання нового запису з тим самим індексом.

Зокрема, під час читання журналу WAL, [логіка перезаписує старі записи новими записами](https://github.com/etcd-io/etcd/blob/release-3.4/wal/wal.go#L448-L462). Таким чином, лише остання версія записів з entry.index &lt;= HardState.commit може вважатися остаточною. Записи з індексом > HardState.commit можуть змінюватися.

"term" у журналі WAL очікуються монотонними.

"indexe" у журналі WAL очікуються:

1. починаються з якогось знімка
2. послідовно зростають після цього знімка, поки вони залишаються у тому ж ‘term’
3. якщо term змінюється, індекс може зменшуватися, але до нового значення, яке є вищим за останній HardState.commit.
4. новий знімок може статися з будь-яким індексом >= HardState.commit, що відкриває нову послідовність для індексів.

{{< figure src="/img/persistent-storage-files-figure-01.png" >}}


### Інструменти {#tools-1}


#### etcd-dump-logs

Журнали WAL etcd можна читати за допомогою інструменту [etcd-dump-logs](https://github.com/etcd-io/etcd/tree/master/tools/etcd-dump-logs):

```sh
% go install go.etcd.io/etcd/v3/tools/etcd-dump-logs@latest

% go run go.etcd.io/etcd/v3/tools/etcd-dump-logs --start-index=0 aname.etcd
```

Зверніть увагу, що:

* Інструмент показує лише записи, а не всі записи WAL (Snapshots, HardStates), які є у файлах журналу WAL.
* Інструмент автоматично застосовує «перезаписи» до записів. Якщо запис був перезаписаний (новішим записом з тим самим індексом), інструмент покаже лише остаточне значення.
* Інструмент також показує непідтверджені записи (з кінця журналу), без інформації про HardState.commitIndex, тому невідомо, чи записи є остаточними чи ні.


## Знімки (Store V2): **member/snap/{term}-{index}.snap** {#snapshots-of-store-v2-membersnapterm-indexsnap}


### Імена файлів: {#file-names-1}

**member/snap/{term}-{index}.snap**

Імена файлів генеруються [тут](https://github.com/etcd-io/etcd/blob/ad5b30297a43daeb5ce7311fa606ce4c1f16618f/server/etcdserver/api/snap/snapshotter.go#L78) `("%016x-%016x.snap") `і використовують 2 шістнадцяткові компоненти:


* term -> Термін Raft (період між виборами) на момент створення знімка
* index -> останньої застосованої пропозиції на момент створення знімка

### Створення {#creation}

Файли *.snap створюються методом [Snapshotter.SaveSnap](https://github.com/etcd-io/etcd/blob/ad5b30297a43daeb5ce7311fa606ce4c1f16618f/server/etcdserver/api/snap/snapshotter.go#L68).

Існує 2 тригери, що контролюють створення цих файлів:


* Новий файл створюється кожні (приблизно) --snapshotCount=(стандартно 100'000) застосовані пропозиції. Це приблизно, оскільки ми можемо отримувати пропозиції пакетами, і ми розглядаємо знімки лише в кінці пакета, нарешті процес знімків асинхронно планується. Назва прапорця (--snapshotCount) є досить оманливою, оскільки вона керує [різницями в значенні індексу між](https://github.com/etcd-io/etcd/blob/ad5b30297a43daeb5ce7311fa606ce4c1f16618f/server/etcdserver/server.go#L1266) останнім індексом знімка та останнім індексом застосованої пропозиції.
* Raft запитує репліку для відновлення зі знімка. Оскільки репліка отримує знімок через мережу (повідомлення msgSnap), вона також зберігає його (легкий) у журналі WAL. Це гарантує, що в кінці журналів WAL завжди є дійсний знімок, за яким слідують записи. Таким чином, це пригнічує потенційну відсутність безперервності у журналах WAL.

Наразі файли приблизно[^3] асоціюються 1-1 із записами знімків журналу WAL. З виведенням з експлуатації сховища v2 ми очікуємо, що файли перестануть записуватися взагалі (опціонально: 3.5.x, обовʼязково 3.6.x).

### Вміст {#content}

Файл містить серіалізований [snapdb.snapshot proto](https://github.com/etcd-io/etcd/blob/ad5b30297a43daeb5ce7311fa606ce4c1f16618f/server/etcdserver/api/snap/snappb/snap.proto#L11) `(uint32 crc, bytes data)`, де:

* у полі 'data' містить [Raftpb.Snapshot](https://github.com/etcd-io/etcd/blob/ad5b30297a43daeb5ce7311fa606ce4c1f16618f/raft/raftpb/raft.proto#L31):
* (байти даних, SnapshotMetadata{index, term, [conf](https://github.com/etcd-io/etcd/blob/ad5b30297a43daeb5ce7311fa606ce4c1f16618f/raft/raftpb/raft.proto#L99)} метадані),

Нарешті, вкладені дані містять серіалізований JSON [вміст сховища v2](#exemplar-json-serialized-store-v2-content-in-etcd-34-snap-files).


Зокрема, є:

* Term
* Index
* Дані про членство:
    * <code>/0/members/8e9e05c52164694d/attributes -> <strong>{\"name\":\"default\",\"clientURLs\":[\"[http://localhost:2379\](http://localhost:2379\)"]}</strong></code>
    * <code>/0/members/8e9e05c52164694d/RaftAttributes -> <strong>"{\"peerURLs\":[\"http://localhost:2380\"]}"</strong></code>
* Версія сховища: /0/version-> 3.5.0

### Інструменти {#tools-2}

#### protoc

Наступна команда дозволяє побачити вміст файлу, коли виконується з кореневого каталогу etcd:

```sh
cat default.etcd/member/snap/0000000000000002-0000000000049425.snap |
  protoc --decode=snappb.snapshot \
    server/etcdserver/api/snap/snappb/snap.proto \
    -I $(go list -f '{{.Dir}}' github.com/gogo/protobuf/proto)/.. \
    -I . \
    -I $(go list -m -f '{{.Dir}}' github.com/gogo/protobuf)/protobuf
```

Аналогічно, ви можете витягти поле 'data' і декодувати як '[Raftpb.Snapshot](https://github.com/etcd-io/etcd/blob/ad5b30297a43daeb5ce7311fa606ce4c1f16618f/raft/raftpb/raft.proto#L31)`'`

### Зразковий вміст сховища v2 у форматі JSON у файлах *.snap версії etcd 3.4: {#exemplar-json-serialized-store-v2-content-in-etcd-34-snap-files}

```json
{
  "Root":{
    "Path":"/",
    "CreatedIndex":0,
    "ModifiedIndex":0,
    "ExpireTime":"0001-01-01T00:00:00Z",
    "Value":"",
    "Children":{
      "0":{
        "Path":"/0",
        "CreatedIndex":0,
        "ModifiedIndex":0,
        "ExpireTime":"0001-01-01T00:00:00Z",
        "Value":"",
        "Children":{
          "members":{
            "Path":"/0/members",
            "CreatedIndex":1,
            "ModifiedIndex":1,
            "ExpireTime":"0001-01-01T00:00:00Z",
            "Value":"",
            "Children":{
              "8e9e05c52164694d":{
                "Path":"/0/members/8e9e05c52164694d",
                "CreatedIndex":1,
                "ModifiedIndex":1,
                "ExpireTime":"0001-01-01T00:00:00Z",
                "Value":"",
                "Children":{
                  "attributes":{
                    "Path":"/0/members/8e9e05c52164694d/attributes",
                    "CreatedIndex":2,
                    "ModifiedIndex":2,
                    "ExpireTime":"0001-01-01T00:00:00Z",
                    "Value":"{\"name\":\"default\",\"clientURLs\":[\"http://localhost:2379\"]}",
                    "Children":null
                  },
                  "RaftAttributes":{
                    "Path":"/0/members/8e9e05c52164694d/RaftAttributes",
                    "CreatedIndex":1,
                    "ModifiedIndex":1,
                    "ExpireTime":"0001-01-01T00:00:00Z",
                    "Value":"{\"peerURLs\":[\"http://localhost:2380\"]}",
                    "Children":null
                  }
                }
              }
            }
          },
          "version":{
            "Path":"/0/version",
            "CreatedIndex":3,
            "ModifiedIndex":3,
            "ExpireTime":"0001-01-01T00:00:00Z",
            "Value":"3.5.0",
            "Children":null
          }
        }
      },
      "1":{
        "Path":"/1",
        "CreatedIndex":0,
        "ModifiedIndex":0,
        "ExpireTime":"0001-01-01T00:00:00Z",
        "Value":"",
        "Children":{


        }
      }
    }
  },
  "WatcherHub":{
    "EventHistory":{
      "Queue":{
        "Events":[
          {
            "action":"create",
            "node":{
              "key":"/0/members/8e9e05c52164694d/RaftAttributes",
              "value":"{\"peerURLs\":[\"http://localhost:2380\"]}",
              "modifiedIndex":1,
              "createdIndex":1
            }
          },
          {
            "action":"set",
            "node":{
              "key":"/0/members/8e9e05c52164694d/attributes",
              "value":"{\"name\":\"default\",\"clientURLs\":[\"http://localhost:2379\"]}",
              "modifiedIndex":2,
              "createdIndex":2
            }
          },
          {
            "action":"set",
            "node":{
              "key":"/0/version",
              "value":"3.5.0",
              "modifiedIndex":3,
              "createdIndex":3
            }
          }
        ]
      }
    }
  }
}
```

## Зміни {#changes}

Цей розділ зарезервовано для опису змін у форматах файлів, введених між різними версіями etcd.

[^1]:
     Метадані сторінки на початку файлу bbolt змінюються на місці.

[^2]:
     Непослідовно, оскільки більшість uint записуються у форматі bigendian

[^3]:
     Початковий (index:0) знімок на початку журналу WAL не асоціюється з файлом *.snap. Також старі файли *.snap (або журнали WAL) можуть бути видалені.

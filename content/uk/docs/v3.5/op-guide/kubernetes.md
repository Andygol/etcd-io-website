---
title: Запуск кластерів etcd як Kubernetes StatefulSet
weight: 4200
description: Запуск etcd як Kubernetes StatefulSet
---

Нижче показано, як виконати [статичний процес завантаження](../clustering/#static) як Kubernetes StatefulSet.

## Приклад маніфесту {#example-manifest}

Цей маніфест містить сервіс та statefulset для розгортання статичного кластера etcd у Kubernetes.

Якщо ви скопіюєте вміст маніфесту у файл з назвою `etcd.yaml`, його можна застосувати до кластера за допомогою цієї команди.

```shell
$ kubectl apply --filename etcd.yaml
```

Після застосування, зачекайте, поки podʼи стануть готовими.

```shell
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
etcd-0   1/1     Running   0          24m
etcd-1   1/1     Running   0          24m
etcd-2   1/1     Running   0          24m
```

Контейнер, використаний у прикладі, включає etcdctl і може бути викликаний безпосередньо всередині podʼів.

```shell
$ kubectl exec -it etcd-0 -- etcdctl member list -wtable
+------------------+---------+--------+-------------------------+-------------------------+------------+
|        ID        | STATUS  |  NAME  |       PEER ADDRS        |      CLIENT ADDRS       | IS LEARNER |
+------------------+---------+--------+-------------------------+-------------------------+------------+
| 4f98c3545405a0b0 | started | etcd-2 | http://etcd-2.etcd:2380 | http://etcd-2.etcd:2379 |      false |
| a394e0ee91773643 | started | etcd-0 | http://etcd-0.etcd:2380 | http://etcd-0.etcd:2379 |      false |
| d10297b8d2f01265 | started | etcd-1 | http://etcd-1.etcd:2380 | http://etcd-1.etcd:2379 |      false |
+------------------+---------+--------+-------------------------+-------------------------+------------+
```

Щоб розгорнути з самопідписним сертифікатом, зверніться до коментованих заголовків конфігурації, що починаються з `## TLS`, щоб знайти значення, які можна розкоментувати. Додаткові інструкції щодо створення сертифіката за допомогою cert-manager наведені в розділі нижче.

```yaml
# file: etcd.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: etcd
  namespace: default
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: etcd
  ##
  ## В ідеалі ми мали б використовувати SRV-записи для виявлення однорангових систем для ініціалізації.
  ## На жаль, виявлення не працюватиме без логіки очікування, поки вони
  ## заповнять контейнер. Цю проблему відносно легко вирішити шляхом
  ## внесенням змін, щоб запобігти запуску процесу etcd до того, як записи
  ## не буде заповнено. У документації до statefulsets коротко описано, як це зробити.
  ##   https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#stable-network-id
  publishNotReadyAddresses: true
  ##
  ## Схема іменування портів клієнта і сервера відповідає схемі, яку використовує etcd
  ## використовує при виявленні за допомогою SRV-записів.
  ports:
  - name: etcd-client
    port: 2379
  - name: etcd-server
    port: 2380
  - name: etcd-metrics
    port: 8080
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: default
  name: etcd
spec:
  ##
  ## Імʼя сервісу встановлюється для того, щоб використовувати сервіс headless.
  ## https://kubernetes.io/docs/concepts/services-networking/service/#headless-services
  serviceName: etcd
  ##
  ## Якщо ви збільшуєте кількість реплік існуючого кластера, вам слід
  ## також оновити прапорець --initial-cluster-state, як зазначено нижче у
  ## конфігурації контейнера.
  replicas: 3
  ##
  ## Для ініціалізації, podʼів etcd повинні бути доступні один одному до того, як
  ## як вони будуть "ready" для трафіку. Політика "Parallel" робить це можливим.
  podManagementPolicy: Parallel
  ##
  ## Для забезпечення доступності кластера etcd, використовується стратегія
  ## rolling update. Для доступності необхідно, щоб принаймні 51% вузлів etcd
  ## онлайн у будь-який момент часу.
  updateStrategy:
    type: RollingUpdate
  ##
  ## Це запит міток поверх podʼів, які мають відповідати кількості реплік.
  ## Він має відповідати міткам шаблону pod. Для отримання додаткової інформації зверніться до
  ## наступної документації:
  ##   https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors
  selector:
    matchLabels:
      app: etcd
  ##
  ## Шаблон для конфігурації pod.
  template:
    metadata:
      ##
      ## Маркування тут привʼязано до “matchLabels" цього StatefulSet та
      ## конфігурації "affinity"  podʼа, що буде створено.
      ##
      ## Ця схема міток у прикладі підходить для одного кластера etcd на
      ## простір імен, але якщо вам потрібно декілька кластерів для одного простору імен,
      ## вам потрібно буде оновити схему міток, щоб вона була унікальною для кожного кластера etcd.
      labels:
        app: etcd
      annotations:
        ##
        ## На нього посилаються у конфігурації контейнера etcd як на частину
        ## імені DNS. Воно має відповідати імені служби, створеної для кластера etcd
        ## Вирішено розмістити його в анотації, а не у параметрах env
        ## пояснюється тим, що на одному кластері etcd має бути лише 1 служба.
        serviceName: etcd
    spec:
      ##
      ## Налаштування спорідненості вузлів необхідне для того, щоб etcd-сервери
      ## не опинилися на одному апаратному забезпеченні.
      ##
      ## Докладнішу інформацію про це наведено у документації з планування:
      ##   https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity
      affinity:
        ## podAntiAffinity - це набір правил для планування, які описують
        ## коли НЕ розміщувати на вузлі pod з цього StatefulSet.
        podAntiAffinity:
          ##
          ## При підготовці до розміщення podʼа на вузлі, планувальник перевірить
          ## на наявність інших podʼів, що відповідають правилам, описаним у labelSelector
          ## відокремлених обраним ключем топології.
          requiredDuringSchedulingIgnoredDuringExecution:
          ## Цей селектор міток шукає app=etcd
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - etcd
            ## Цей ключ топології позначає спільну мітку, яка використовується на вузлах у
            ## кластері. Конфігурація podAntiAffinity передбачає, що
            ## якщо інший pod має мітку app=etcd на вузлі, то
            ## планувальник не повинен розміщувати на цьому вузлі інший pod.
            ##   https://kubernetes.io/docs/reference/labels-annotations-taints/#kubernetesiohostname
            topologyKey: "kubernetes.io/hostname"
      ##
      ## Контейнери в podʼі
      containers:
      ## У цьому прикладі є тільки цей контейнер etcd.
      - name: etcd
        image: quay.io/coreos/etcd:v3.5.15
        imagePullPolicy: IfNotPresent
        ports:
        - name: etcd-client
          containerPort: 2379
        - name: etcd-server
          containerPort: 2380
        - name: etcd-metrics
          containerPort: 8080
        ##
        ## Ці проби не зможуть пройти через TLS для самопідписних сертифікатів, тому etcd
        ## налаштовано на передачу метрик через порт 8080 і далі.
        ##
        ## Як зазначено на сторінці «Моніторинг etcd», /readyz та /livez було
        ## додано у версії 3.5.12. До цього моніторинг вимагав додаткового інструментарію
        ## всередині контейнера, щоб змусити ці проби працювати.
        ##
        ## Значення у цій перевірці готовності слід додатково перевірити,
        ## це лише приклад конфігурації.
        readinessProbe:
          httpGet:
            path: /readyz
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 30
        ## Значення у цій конфігурації слід додатково перевірити, це
        ## це лише приклад конфігурації.
        livenessProbe:
          httpGet:
            path: /livez
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        env:
        ##
        ## Змінні оточення, визначені тут, можуть бути використані іншими частинами
        ## конфігурації контейнера. Вони інтерпретуються Kubernetes, а не
        ## у середовищі контейнера.
        ##
        ## Ці змінні env передають інформацію про контейнер.
        - name: K8S_NAMESPACE
          valueFrom:
            fieldRef:
             fieldPath: metadata.namespace
        - name: HOSTNAME
          valueFrom:
            fieldRef:
             fieldPath: metadata.name
        - name: SERVICE_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.annotations['serviceName']
        ##
        ## Налаштування etcdctl всередині контейнера для підключення до вузла etcd
        ## у контейнері зменшує плутанину під час налагодження.
        - name: ETCDCTL_ENDPOINTS
          value: $(HOSTNAME).$(SERVICE_NAME):2379
        ##
        ## Конфігурація TLS-клієнта для etcdctl у контейнері.
        ## Шляхи до цих файлів є частиною монтування тому "etcd-client-certs".
        # - name: ETCDCTL_KEY
        #   value: /etc/etcd/certs/client/tls.key
        # - name: ETCDCTL_CERT
        #   value: /etc/etcd/certs/client/tls.crt
        # - name: ETCDCTL_CACERT
        #   value: /etc/etcd/certs/client/ca.crt
        ##
        ## Використовуйте це значення URI_SCHEME для не-TLS кластерів.
        - name: URI_SCHEME
          value: "http"
        ## TLS: Використовуйте цей URI_SCHEME для кластерів TLS.
        # - name: URI_SCHEME
        # value: "https"
        ##
        ## Якщо ви використовуєте інший контейнер, виконуваний файл може знаходитися у
        ## іншому місці. У цьому прикладі використано повний шлях, щоб усунути
        ## двозначності для вас, читача.
        ## Часто замість "/usr/local/bin/etcd" можна просто використати "etcd", і це
        ## працюватиме, оскільки у $PATH є тека, яка містить "etcd".
        command:
        - /usr/local/bin/etcd
        ##
        ## Аргументи, що використовуються з командою etcd всередині контейнера.
        args:
        ##
        ## Вкажіть імʼя сервера etcd.
        - --name=$(HOSTNAME)
        ##
        ## Налаштуйте etcd на використання постійного сховища, налаштованого нижче.
        - --data-dir=/data
        ##
        ## У цьому прикладі ми консолідуємо WAL для спільного використання простору з
        ## текою даних. Це не є ідеальним варіантом у промислових середовищах, і
        ## слід розміщувати у власному томі.
        - --wal-dir=/data/wal
        ##
        ## Конфігурації URL-адрес тут параметризовано, і вам не потрібно
        ## нічого з ними робити.
        - --listen-peer-urls=$(URI_SCHEME)://0.0.0.0:2380
        - --listen-client-urls=$(URI_SCHEME)://0.0.0.0:2379
        - --advertise-client-urls=$(URI_SCHEME)://$(HOSTNAME).$(SERVICE_NAME):2379
        ##
        ## Для початкового завантаження кластера має бути встановлено значення "new". Для масштабування
        ## кластера, цей параметр слід змінити на "existing", коли кількість реплік
        ## збільшується. Якщо параметр встановлено неправильно, etcd спробує
        ## запуск, але беспечно зазнає невдачі.
        - --initial-cluster-state=new
        ##
        ## Токен, який використовується для ініціалізації кластера. Для цього рекомендується
        ## використовувати унікальний токен для кожного кластера. У цьому прикладі задано параметр
        ## на унікальність для простору імен, але якщо ви розгортаєте декілька кластерів etcd
        ## у тому самому просторі імен, вам слід зробити щось додаткове, щоб
        ## щоб забезпечити унікальність серед кластерів.
        - --initial-cluster-token=etcd-$(K8S_NAMESPACE)
        ##
        ## Початковий прапорець кластера потрібно оновити відповідно до кількості
        ## налаштованих реплік. У поєднанні вони трохи важко читаються.
        ## Ось як виглядає один параметризований одноранговий кластер:
        ##   etcd-0=$(URI_SCHEME)://etcd-0.$(SERVICE_NAME):2380
        - --initial-cluster=etcd-0=$(URI_SCHEME)://etcd-0.$(SERVICE_NAME):2380,etcd-1=$(URI_SCHEME)://etcd-1.$(SERVICE_NAME):2380,etcd-2=$(URI_SCHEME)://etcd-2.$(SERVICE_NAME):2380
        ##
        ## Прапорець peer urls має бути без змін.
        - --initial-advertise-peer-urls=$(URI_SCHEME)://$(HOSTNAME).$(SERVICE_NAME):2380
        ##
        ## Це дозволяє уникнути збою проби, якщо ви вирішите налаштувати TLS.
        - --listen-metrics-urls=http://0.0.0.0:8080
        ##
        ## Ось деякі конфігурації, які ви можете спробувати увімкнути, але
        ## варто вивчити докладніше, щоб визначити, які налаштування найкраще підходять саме вам.
        # - --auto-compaction-mode=periodic
        # - --auto-compaction-retention=10m
        ##
        ## Конфігурація TLS-клієнта для etcd, повторне використання змінних etcdctl env.
        # - --client-cert-auth
        # - --trusted-ca-file=$(ETCDCTL_CACERT)
        # - --cert-file=$(ETCDCTL_CERT)
        # - --key-file=$(ETCDCTL_KEY)
        ##
        ## Налаштування сервера TLS для etcdctl у контейнері.
        ## Шляхи до цих файлів є частиною монтування тому "etcd-server-certs".
        # - --peer-client-cert-auth
        # - --peer-trusted-ca-file=/etc/etcd/certs/server/ca.crt
        # - --peer-cert-file=/etc/etcd/certs/server/tls.crt
        # - --peer-key-file=/etc/etcd/certs/server/tls.key
        ##
        ## Це конфігурація монтування.
        volumeMounts:
        - name: etcd-data
          mountPath: /data
        ##
        ## Конфігурація TLS-клієнта для etcdctl
        # - name: etcd-client-tls
        #   mountPath: "/etc/etcd/certs/client"
        #   readOnly: true
        ##
        ## Конфігурація сервера TLS
        # - name: etcd-server-tls
        #   mountPath: "/etc/etcd/certs/server"
        #   readOnly: true
      volumes:
      ##
      ## Конфігурація TLS-клієнта
      # - name: etcd-client-tls
      #   secret:
      #     secretName: etcd-client-tls
      #     optional: false
      ##
      ## Конфігурація сервера TLS
      # - name: etcd-server-tls
      #   secret:
      #     secretName: etcd-server-tls
      #     optional: false
  ##
  ## Цей StatefulSet використовує поле volumeClaimTemplate для створення PVC у
  ## кластері для кожної репліки. Ці PVC не можуть бути легко змінені пізніше.
  volumeClaimTemplates:
  - metadata:
      name: etcd-data
    spec:
      accessModes: ["ReadWriteOnce"]
      ##
      ## У деяких кластерах потрібно явно задавати клас сховища.
      ## У цьому прикладі буде використано стандартний клас сховища.
      # storageClassName: ""
      resources:
        requests:
          storage: 1Gi
```

## Генерація сертифікатів {#generating-certificates}

У цьому розділі ми використовуємо [Helm](https://helm.sh) для встановлення оператора з назвою [cert-manager](https://cert-manager.io/).

З встановленим cert-manager у кластері можна генерувати самопідписні сертифікати в кластері. Ці згенеровані сертифікати розміщуються всередині обʼєкта секрету, який можна прикріпити як файли в контейнерах.

Це команда helm для встановлення cert-manager.

```shell
$ helm upgrade --install --create-namespace --namespace cert-manager cert-manager cert-manager --repo https://charts.jetstack.io --set crds.enabled=true
```

Це приклад конфігурації ClusterIssuer для генерації самопідписних сертифікатів.

```yaml
# file: issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned
spec:
  selfSigned: {}
```

Цей маніфест створює обʼєкти сертифікатів для клієнтських і серверних сертифікатів, посилаючись на ClusterIssuer "selfsigned". dnsNames повинні бути вичерпним списком дійсних імен хостів для сертифікатів, які створює cert-manager.

```yaml
# file: certificates.yaml
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: etcd-server
  namespace: default
spec:
  secretName: etcd-server-tls
  issuerRef:
    name: selfsigned
    kind: ClusterIssuer
  commonName: etcd
  dnsNames:
  - etcd
  - etcd.default
  - etcd.default.svc.cluster.local
  - etcd-0
  - etcd-0.etcd
  - etcd-0.etcd.default
  - etcd-0.etcd.default.svc
  - etcd-0.etcd.default.svc.cluster.local
  - etcd-1
  - etcd-1.etcd
  - etcd-1.etcd.default
  - etcd-1.etcd.default.svc
  - etcd-1.etcd.default.svc.cluster.local
  - etcd-2
  - etcd-2.etcd
  - etcd-2.etcd.default
  - etcd-2.etcd.default.svc
  - etcd-2.etcd.default.svc.cluster.local
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: etcd-client
  namespace: default
spec:
  secretName: etcd-client-tls
  issuerRef:
    name: selfsigned
    kind: ClusterIssuer
  commonName: etcd
  dnsNames:
  - etcd
  - etcd.default
  - etcd.default.svc.cluster.local
  - etcd-0
  - etcd-0.etcd
  - etcd-0.etcd.default
  - etcd-0.etcd.default.svc
  - etcd-0.etcd.default.svc.cluster.local
  - etcd-1
  - etcd-1.etcd
  - etcd-1.etcd.default
  - etcd-1.etcd.default.svc
  - etcd-1.etcd.default.svc.cluster.local
  - etcd-2
  - etcd-2.etcd
  - etcd-2.etcd.default
  - etcd-2.etcd.default.svc
  - etcd-2.etcd.default.svc.cluster.local
```

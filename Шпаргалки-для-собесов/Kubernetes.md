# Шпаргалка по Kubernetes

## Что такое Kubernetes?

**Kubernetes** - инфраструктурная платформа, содержащая в себе фреймворк для декларативного управления конфигурациями приложений на основе контейнеризации и виртуализации.
Она является де-факто стандартом для оркестрации контейнеров, предоставляя богатый функционал для управления инфраструктурой, сетями, хранением данных и приложениями.

## Что такое POD?

**Pod** - минимальная единица развертывания в Kubernetes, которая предоставляет собой группу контейнеров, объединенных общей сетью(общий localhost, общий внешний IP) и общими ресурсами.
Контейнеры внутри POD работают в тесной связке.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
  - name: php-fpm
    image: php:fpm
```

### Для чего при старте PODа создается контейнер с процессом pause?

Для того чтобы создать сетевую изоляцию Kubernetes при старте POD'а запускает специальный контейнер `pause`.
Он отвечает за управление сетью и обеспечивает контейнерам внутри POD'а возможность взаимодействовать через localhost.
Он не выполняет реальную работу, а является посредником при сетевом взаимодействии.

### Как можно запускать поды?

POD'ы обычно запускаются с помощью **workloads**, которые обеспечивают управление жизненным циклом POD'ов.

Примеры:

1. ***(ReplicationController)*** **Deployment** - для развертывания и управления репликами POD'ов с возможность откатов.
2. **StatefulSet** - для приложений с состоянием, которые требуют уникальности и постоянства идентификаторов.
3. **DaemonSet** - для запуска одного POD'а на каждом узле кластера.
4. **Job** - для запуска одноразовых задач.
5. **CronJob** - для запуска задач по расписанию.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
```

## В чем разница между Deployment и StatefulSet?

1. В именах: у первого некий случайный хэш, у второго - порядковый номер.
2. В работе с дисками: StatefulSet дожидается пока POD с таким же именем завершит работы, чтобы занять тот диск, который был к нему привязан.
3. В стратегиях перезапуска POD'ов при обновлении.

## Что такое ReplicaSet?

**ReplicaSet** - это объект, который управляет количеством реплик POD'ов в Kubernetes.
ReplicaSet гарантирует, что в любой момент времени будет существовать определенное количество копий POD'ов.
В Kubernetes обычно используют Deployment, который управляет ReplicaSet.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
```

## Что такое Kubernetes probes?

Проверки состояния контейнера - механизмы для мониторинга состояния приложений и контейнеров.

Kubernetes поддерживает три вида проверок:

1. **Startup probe** - проверка, которая используется для определения, когда контейнер полностью инициализировался. Если эта проверка не проходит, другие пробы не выполняются.
2. **Readiness probe** - проверка готовности контейнера к обслуживанию трафика.
Если она не проходит, контейнер не будет получать трафик.
3. **Liveness probe** - проверка живости контейнера.
Если она не проходит, контейнер перезапускается.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:v1
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /readiness
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

### Какие хорошие практики существуют при создании проб?

- Пробы должны проверять рабочие и критические компоненты приложения, чтобы гарантировать, что приложение может обрабатывать запросы пользователя.
- Пробы не должны быть слишком частыми или тяжелыми, чтобы не создавать дополнительную нагрузку на систему.
- Пробы не должны зависеть от внешних сервисов, если только это не критично для работы приложения.
- Используйте сквозные проверки, если приложение имеет несколько взаимосвязанных сервисов.

## Что такое Pod Disruption Budget?

**Pod Disruption Budget (PDB)** - механизм, который гарантирует, что в кластере всегда будет доступно минимальное количество POD'ов для определенного приложения.
PDB помогает предотвратить слишком большое количество перерывов в работе приложения при обслуживании или удалении POD'ов.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

## Что такое priority classes?

**Priority Classes** - механизм, который позволяет назначать приоритеты POD'ам, чтобы Kubernetes мог управлять распределением ресурсов между POD'ами с разными приоритетами, особеннос в случае нехватки ресурсов.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class is for critical workloads"
```

## Что такое POD eviction?

**Eviction** - процесс удаления POD'ов с узлов, когда ресурсы на узле становятся дефицитными(нехватки памяти или диска).
Kubernetes может эвакуировать POD'ы с низким приоритетом, чтобы освободить ресурсы для более важных приложений.

```sh
kubectl drain <node-name> --ignore-daemonsets
```

## Каким образом можно управлять размещением PODов на конкретных нодах кластера k8s?

1. **NodeSelector/Node Affinity** - позволяет указать, на каких узлах можно запускать POD'ы, основываясь на метках нод. 
2. **Taints/Tolerations** - механизм, который запрещает или разрешает запуск POD'ов на определенных узлах в зависимости от их меток. 
3. **Pod Affinity/Anti-Affinity** - механизм, позволяющий задавать правила для размещения POD'ов на одних или разных узлах в зависимости от меток.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: "disktype"
            operator: In
            values:
            - ssd
  containers:
  - name: myapp
    image: myapp:v1
```

## Каким образом можно управлять вычислительными ресурсами в k8s?

**Resources requests** и **limits** позволяют управлять количеством ресурсов, которое контейнер может использовать.

- **Request** - минимальное количество ресурсов, которое контейнер гарантировано получит.
- **Limits** - максимальное количество ресурсов, которое контейнер может использовать.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: myapp
    image: myapp:v1
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

## Что такое Kubernetes service?

**Service** - абстракция, которая позволяет организовывать доступ к приложениям, работающим в Kubernetes, через постоянный IP-адрес или DNS-имя.
Сервис может выполнять балансировку трафика между POD'ами.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

## Каким образом можно предоставить приложение, которые работает в кластере, пользователям?

Если приложение работает по HTTP, то можно использовать `Ingress-контроллер`.
Ingress позволяет маршрутизировать HTTP/HTTPS-трафик от пользователей к приложениям в кластере.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

Если приложение работает по бинарному протоколу(PostgreSQL), то обычно используют Service с типом **NodePort** или **LoadBalancer**, чтобы сделать его доступным извне.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  type: NodePort
  ports:
    - port: 5432
      targetPort: 5432
      nodePort: 30000
  selector:
    app: postgres
```

### Почему Ingress удобен?

Используя Ingress, можно управлять маршрутизацией трафика для всех приложений внутри кластера через одну точку входа.
Это позволяет эффективно управлять HTTP/HTTPS-трафиком, используя такие функции как:

- Балансировка нагрузки
- Поддержка HTTPS
- Маршрутизация по поддоменам или URL
- Поддержка канареечных развертываний

## Каким образом организована сеть в k8s?

В Kubernetes существует три типа сети:

- **Node Network** - сеть, в которую объединены ноды.
В зависимости от использования CNI-плагина, ноды могут работать только в одной подсети, либо в нескольких.
- **Pod Network** - сеть, в которой получают IP-адреса запускаемые PODы.
- **Service Network** - сеть, в которой получают адреса Kubernetes services.

Pod network и service network орнанизуются при помощи так называемых CNI-плагинов.


## Что такое CNI-плагин и для чего он нужен?

CNI расшифровывается как **Container Network Interface**.
Он представляет собой некий уровень абстракции над реализацией сети.
Мы может работать с верхнеуровневыми абстракциями вроде "IP-адрес POD'а", Endpoint.
За то как это будет реализовано на физическом уровне отвечают CNI-плагины, которые реализуют разный функционал и показывают разную сетевую производительность.

Примеры CNI-плагинов:

- [Flannel](https://github.com/flannel-io/flannel)
- [Calico](https://github.com/projectcalico/calico)
- [Cilium](https://github.com/cilium/cilium)

## Что такое EGRESS?

**EGRESS** - возможность назначить внешний IP-адрес для исходящего за пределы кластера k8s трафика приложений.
Поддержка EGRESS должна быть реализована на уровне CNI-плагина и может быть описана специальным объектом на уровне неймспейса.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
```

## Как мы можем ограничить трафик в Kubernetes?

Для ограничения трафика от приложений используется объект **NetworkPolicy**.
При помощи него мы можем ограничивать входящий и исходящий трафик на уровне неймспейса и компонентов, описанных в нем.
Его поддержка должна быть реализована на уровне CNI-плагина.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-specific
  namespace: my-namespace
spec:
  podSelector:
    matchLabels:
      app: my-app
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              role: frontend
```

## Что такое Service Mesh и для чего он нужен?

**Service Mesh** - паттерн, позволяющий более гибко управлять трафиком в Kubernetes.
Servise Mesh состоит из:
- *control plane*, который собирает информацию о кластере k8s
- запущенных в нем приложениях
- дополнительных объектов, которые могут быть описаны для дополнительной конфигурации
- sidecar-контейнеров, которые инжектятся в POD'ы автоматически при помощи *mutation webhook*.

**Sidecar-контейнеры**  - reverse-прокси, перехватывающие входящий и исходящий трафик приложений в контейнерах и управляющие им в зависимости от полученной из *control plane* конфигурации.

Основные задачи для Service Mesh:

- Гарантий доставки трафика до приложений(retry, timeouts, circuit breaker)
- Увеличение безопасности при помощи шифрования трафика, верификации трафика на основе сертификатов, реализация дополнительных правил и разрешений на передачу трафика
- Реализация переключения трафика при *canary*, *a/b*, *blue/green* стратегий деплоя.

Недостатки:

- Избыточная сложность
- Повышенная ресурсоемкость
- Накладные расходы

## В чем отличие меток от аннотаций?

**Метки** используются для реализации механизмов поиска и группировки объектов.
**Аннотации** используются для описания метаинформации на объекте.

## Из каких компонентов состоит k8s и каково их назначение?

Kubernetes состоит из *control plane* и *data plane*.

**Control Plane** - управляющий контур, который запускается в пределах нод, называемых *master*,
и может запускаться как одиночном режиме(*single master*), так и в распределенном(*multi master*).

Control plane содержит:

- **ETCD** - хранилище конфигурации кластера
- **Kubernetes API** - предоставляет API, посредством которого взаимодействуют компоненты k8s,
а также клиенты находящиеся внутри и вне кластера.
- **Kubernetes Controller Manager** - реализует концепцию контроллеров, которые управляют базовыми сущностями кластеров.
- **Kubernetes scheduler** - выбирает ноды, на которых нужно запускать PODы.
- **Cloud controller manager** - используется для реализации функций работы с облаком.

**Data plane** - компоненты, которые запущены на каждой ноде.

- **kubelet** - следит за изменениями конфигурации ноды, применяет конфигурации, делает пробы контейнеров, отчитывается о статусе контейнеров, работает с CRI-плагином и реализует функции запуска и остановки контейнеров.
- **kube-proxy** - отвечает за сетевой компонент. Работает с CNI-плагином и обеспечивает функционрирование сущности *Service* в пределах своей ноды.

## Что такое kube-proxy и для чего он нужен?

**kube-proxy** - компонент *data plane*, который работает на каждой ноде.
Он взаимодействеут с CNI-плагином, обеспечивая функционирование *pod network*.
А также обеспечивает функционирование описанных в кластере сервисов в пределах своей ноды.
В зависимости от режима выступает либо в роли прокси, либо как контроллер правил IPTABLES/IPVS.

## Что такое CRI?

CRI расшифровыется как **Container Runtime Interface**.
Это спецификация, описывающая некий уровень абстракции, который позволяет унифицированно использовать разные версии ПО для работы с контейнерами, например *containerd* или *CRI-O*.

## Какие типы volum'ов можно использовать в Kubernetes?

1. hostpath, но тогда POD должен быть привязан к ноде.
2. local-storage - автоматически привязывает POD, который его использует, к нужной ноже.

Также можно использовать сетевые диски при помощи CSI-плагинов.

## Что такое CSI-плагин?

CSI расшифровывается как **Container Storage Interface**.
Это абстракция, позволяющая унифицированно использовать сетевые файловые системы, построенные на разных технологических базах.

Мы описываем *storageClass*, соответствующий дискам определенного типа, и деплоим в кластер *provisioner*.

**Provisioner** - специальное ПО, которое может заказывать сетевые диски в системе, способной их предоставлять.

Далее мы описываем объект *persistentVolumeClaim* с указанием нужного *storageClass*.

Provisioner при появлении *PVC* заказывает диск нужного размера в системе, которая их предоставляет, создает объект *persistentVolume* и привязывает его к *PVC*.
Когда происходит запуск POD'а на ноде, соотвествующий диск монтируется на нужную ноду по определенному пути, и этот путь монтируется на файловую систему PODа.

```yaml
# Storage Class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2

---
# PVC
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: fast
```

## Каким образом мы можем разделить права на Kubernetes?

Для разделения прав в Kubernetes применяется механизм **Role Based Access Control(RBAC)**.
В рамках него есть три группы сущностей:

- *user* или *service account* - который описывает субъект доступа.
- *role* или *cluster role* - который описывает разрешения.
- *roleBinding* или *clusterRoleBinding* - который привязывает списки разрешений к субъекту.

```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-namespace
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]

---
# Role Binding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: my-namespace
subjects:
  - kind: User
    name: example-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### В чем отличие service account от user?

**User** не имеет записей в Kubernetes API, управление осуществляется внешними механизмами.
Они предназначены для событий вне кластера.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: my-namespace
```

## Какие механизмы аутентификации используются в Kubernetes?

Kubernetes может использовать:

- сертификаты X509
- Bearer-токены
- аутентифицирующий прокси
- HTTP Basic Auth

При помощи этих механизмов можно реализовывать большое количество схем авторизации: от статичного файла с паролями до OpenID OAuth2.

Более того, допускается применение нескольких схем авторизации одновременно.
По умолчанию используются:

- service account tokens - для Service Accounts
- X509 - для Users

## Что такое namespace в k8s и для чего он нужен?

Участвует в формировании DNS-имен внутри кластера. `[service name].[namespace].[суффикс кластера]`.

Суффикс по-умолчанию: *svc.cluster.local*.

## Что такое финалайзеры и для чего они нужны?

Это специальные ключи в манифесте объекта, описывающие действия, которые требуется совершить до удаления обзъекта.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
  namespace: my-namespace
  finalizers:
    - my-finalizer.kubernetes.io
data:
  key: value
```

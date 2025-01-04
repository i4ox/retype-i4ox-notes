# Шпаргалка по Helm

## Что такое Helm и зачем он нужен?

**Helm** - менеджер пакетов для Kubernetes, который позволяет:

- Описывать приложения и их конфигурации в виде *charts*.
- Устанавливать и управлять приложениями с помощью командной строки.
- Упрощать обновление, удаление и версионирование приложений.

## Что такое Helm Chart?

**Chart** - набор файлов, описывающих Kubernetes-приложение. Он включает:

- Шаблоны манифестов
- Файлы конфигурации
- Метаинформацию

Пример структуры:

```txt
my-chart/
├── Chart.yaml        # Метаданные chart'а
├── values.yaml       # Значения по умолчанию
├── templates/        # Шаблоны манифестов
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── charts/           # Зависимости (другие charts)
└── README.md         # Документация
```

## Как создать Helm Chart?

```sh
helm create my-chart
```

## Как установить приложение с помощью Helm?

```sh
# helm install <release-name> <chart-path-or-repo>
helm install my-app ./my-chart
```

## Что такое Release?

**Release** - установленная версия chart'а в кластере.
Имя релиза позволяет управлять установленным приложением.

## Как просмотретьт установленные Releases?

```sh
helm list
```

## Как удалить Release?

```sh
helm uninstall <release-name>
```

## Что такое Values?

**Values** - настройки, которые влияют на генерацию манифестов.
Они хранятся в *values.yaml* и могут быть переопределены при установке.

```yaml
replicaCount: 3
image:
  repository: nginx
  tag: "1.21.0"
service:
  type: ClusterIP
  port: 80
```

Способ через переопределение значений:

```sh
helm install my-app ./my-chart --set replicaCount=2
```

## Как работают шаблоны в Helm?

Шаблоны пишутся на языке *Go Templates* и используют директивы.

```helm
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-{{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

## Как проверить сгенерированные манифесты?

```sh
helm template <chart-path>
```

## Как обновить Release?

```sh
# helm upgrade <release-name> <chart-path-or-repo>
helm upgrade my-app ./my-chart
```

## Что такое Helm Repository?

**Helm Repository** - хранилище Helm Chart'ов.

Для работы с репозиториями есть команды.

```sh
helm repo add <repo-name> <repo-url>
helm repo update
helm repo list
```

Пример:

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

## Что такое Helm Hooks?

**Helm Hooks** - специальные шаблоны, которые выполняются на разных этапах жизненного цикла.

```helm
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ .Release.Name }}-post-install
  annotations:
    "helm.sh/hook": post-install
spec:
  template:
    spec:
      containers:
      - name: job
        image: busybox
        command: ["echo", "Hello, world!"]
      restartPolicy: Never
```

## Как протестировать Helm Chart?

```helm
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Release.Name }}-test
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["true"]
```

```sh
helm test <release-name>
```

## Как откатить изменения?

```sh
# helm rollback <release-name> <revision>
helm rollback my-app 1
```

## Как работают зависимости в Helm?

```yaml
# Chart.yaml
dependencies:
  - name: redis
    version: 14.8.8
    repository: https://charts.bitnami.com/bitnami
```

```sh
helm dependency update
```

## Что такое Helm Secrets?

Для безопасного хранения секретов можно использовать плагин *Helm Secrets*, который шифрует файлы через *SOPS*.

```sh
sops -e values.secret.yaml > values.secret.enc.yaml
```

```sh
helm install my-app ./my-chart -f values.secret.enc.yaml --decrypt
```

## Как настроить CI/CD в Helm?

```yaml
stages:
  - deploy

deploy:
  stage: deploy
  image: alpine/helm:3.12.3
  script:
    - helm repo add bitnami https://charts.bitnami.com/bitnami
    - helm upgrade --install my-app ./my-chart
```

## Полезные команды Helm

```sh
helm history <release-name>
helm lint ./my-chart
helm uninstall <release-name> --purge
helm template ./my-chart
```

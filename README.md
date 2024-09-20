# Домашнее задание к занятию «Helm» - Подус Сергей

### Задание 1. Подготовить Helm-чарт для приложения

1. Необходимо упаковать приложение в чарт для деплоя в разные окружения. 
2. Каждый компонент приложения деплоится отдельным deployment’ом или statefulset’ом.
3. В переменных чарта измените образ приложения для изменения версии.

------
### Задание 2. Запустить две версии в разных неймспейсах

1. Подготовив чарт, необходимо его проверить. Запуститe несколько копий приложения.
2. Одну версию в namespace=app1, вторую версию в том же неймспейсе, третью версию в namespace=app2.
3. Продемонстрируйте результат.

## Ответ:

### Задание 1.

1. Версия helm:

![Scrin 1](https://github.com/Wanderwille/scrinshot/blob/scrin2/kub_10/helm%20version.png)

Попробуем создать шаблон:

```yaml
root@User:/home/test/kub_10/src# helm template helm_chart_nginx
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
---
# Source: test/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: main
  labels:
    app: main
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: main
---
# Source: test/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
        - name: test
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

Изменим версию приложения в Chart.yaml на "1.18.0":

```yaml
root@User:/home/test/kub_10/src# helm template helm_chart_nginx
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
---
# Source: test/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: main
  labels:
    app: main
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: main
---
# Source: test/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
        - name: test
          image: "nginx:1.18.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

Версия изменилась на "1.18.0"

Теперь же попробуем задать версию в файле переменных (values.yaml)

```yaml
root@User:/home/test/kub_10/src# helm template helm_chart_nginx
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
---
# Source: test/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: main
  labels:
    app: main
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: main
---
# Source: test/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: main
  labels:
    app: main
spec:
  replicas: 1
  selector:
    matchLabels:
      app: main
  template:
    metadata:
      labels:
        app: main
    spec:
      containers:
        - name: test
          image: "nginx:1.20.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
```

Как видим, версия изменилась на ту, которая была задана в файле переменных ```image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"``` в этой строчке указано, что в первую очередь данные брать из файла с переменными, если в них не будет указано данных, тогда подставлять данные из файла Chart.yaml

### Задание 2. 

1. Проверяем работу чарт'a: 

![Scrin](https://github.com/Wanderwille/scrinshot/blob/scrin2/kub_10/helm%20install.png)

Проверим еще раз, создав несколько реплик приложения:

![Scrin](https://github.com/Wanderwille/scrinshot/blob/scrin2/kub_10/helm%20install%202.png)

2. Создадим чарт в namespaces app1:

![Scrin](https://github.com/Wanderwille/scrinshot/blob/scrin2/kub_10/app1.png)

При попытке создания второй версии, получаем ошибку:
```bash
root@User:/home/test/kub_10/src# helm install test2 --namespace app1 helm_chart_nginx
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /root/.kube/config
Error: INSTALLATION FAILED: Unable to continue with install: Service "main" in namespace "app1" exists and cannot be imported into the current release: invalid ownership metadata; annotation validation error: key "meta.helm.sh/release-name" must equal "test2": current value is "test1"
```
Это значит, что "Служба "main" в пространстве имен "app1" существует и не может быть импортирована в текущую версию"

Изменяем файл deployment.yaml и файл services.yaml

Меняем в файлах ```main => main2```

Результат:

![Scrin](https://github.com/Wanderwille/scrinshot/blob/scrin2/kub_10/app1-2.png)

Запускаем третью версию во втором namespaces:

![Scrin](https://github.com/Wanderwille/scrinshot/blob/scrin2/kub_10/app2.png)



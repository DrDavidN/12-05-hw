# «Сетевое взаимодействие в K8S. Часть 2» - Дрибноход Давид

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.

#### Ответ:

Создал отдельный namespace ``` kubectl create namespace 12-05-hw ```

Создал deployment_front.yaml и применил его

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/33350479-1218-4689-a092-eaf3a4404391)

2. Создать Deployment приложения _backend_ из образа multitool. 

#### Ответ:

Создал deployment_back.yaml и применил его

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/a9a62775-d681-41e9-894d-4553b9e80dcb)

Проверил статус

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/967128fc-6aad-4880-8214-a8170330f924)

3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 

#### Ответ:

Создал service.yaml, применил его и проверил

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/d520b144-d7c1-4f31-a5d6-69b5b77b759c)

4. Продемонстрировать, что приложения видят друг друга с помощью Service.

#### Ответ:

Проверил доступность используя curl

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/172817c0-f040-4b4c-a877-03dd38078ae6)

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/2b5b7e2b-c797-499b-a717-927399e06bf1)

5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

#### Ответ:

deployment-front.yaml

``` YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: 12-05-hw
  labels:
    app: front
spec:
  selector:
    matchLabels:
      app: front
  replicas: 3
  template:
    metadata:
      labels:
        app: front
        component: network2
    spec:
      containers:
      - name: nginx
        image: nginx:1.25.4
        ports:
        - containerPort: 80
```

deployment-back.yaml

``` YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: 12-05-hw
  labels:
    app: back
spec:
  selector:
    matchLabels:
      app: back
  replicas: 1
  template:
    metadata:
      labels:
        app: back
        component: network2
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 8080
        env:
          - name: HTTP_PORT
            value: "8088"
```

service.yaml

``` YAML
apiVersion: v1
kind: Service
metadata:
  name: frontback-service
  namespace: 12-05-hw
  labels:
    component: network2
spec:
  selector:
    component: network2
  ports:
    - protocol: TCP
      name: nginx
      port: 9001
      targetPort: 80
    - protocol: TCP
      name: multitool
      port: 9002
      targetPort: 8088
```

------

### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.

#### Ответ:

Проверяю что Ingress-controller запущен

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/b303de27-2622-4612-8e29-1a661d0ec1d5)

2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.

#### Ответ:

Создал ingress.yaml, применил его и проверил

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/d6412f33-9c62-400e-a577-d2e34f0a225c)

3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.

#### Ответ:

``` curl http://myingress.ru ```

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/b6f0b056-45c6-4ecd-b124-3cc9c181e697)

``` curl http://myingress.ru/api ```

![image](https://github.com/DrDavidN/12-05-hw/assets/128225763/417f19a9-663f-44ae-a697-a3d2c017ebaa)


4. Предоставить манифесты и скриншоты или вывод команды п.2.

#### Ответ:

``` YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  namespace: 12-05-hw
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myingress.ru
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontback-service
            port:
              number: 9001
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: frontback-service
            port:
              number: 9002
```

------

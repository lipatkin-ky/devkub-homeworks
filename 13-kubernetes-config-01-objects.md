# Домашнее задание к занятию "13.1 контейнеры, поды, deployment, statefulset, services, endpoints"
Настроив кластер, подготовьте приложение к запуску в нём. Приложение стандартное: бекенд, фронтенд, база данных. Его можно найти в папке 13-kubernetes-config.

---
#### Сборка
```
% docker-compose up --build
Building frontend
[+] Building 95.2s (20/20) FINISHED  
...
...
[+] Building 32.1s (12/12) FINISHED
...
...
Creating 13-kubernetes-config_frontend_1 ... done
Creating 13-kubernetes-config_db_1       ... done
Creating 13-kubernetes-config_backend_1  ... done
...
...
```
#### Просматриваю наличие образов
```
% docker images
REPOSITORY                      TAG         IMAGE ID       CREATED         SIZE
13-kubernetes-config_backend    latest      7b173917cdb5   3 minutes ago   1.07GB
13-kubernetes-config_frontend   latest      4b86ba5f6609   3 minutes ago   142MB
postgres                        13-alpine   928a7a35a1ad   3 days ago      207MB
docker/getting-started          latest      bd9a9f733898   8 weeks ago     28.8MB
```
#### Подготовка к отправке на Docker-HUB
```
% docker tag 13-kubernetes-config_backend:latest constantinelipatkin/backend:13.1
% docker tag 13-kubernetes-config_frontend:latest constantinelipatkin/frontend:13.1
%
% docker images | grep 13.1
constantinelipatkin/backend     13.1        7b173917cdb5   17 hours ago   1.07GB
constantinelipatkin/frontend    13.1        4b86ba5f6609   17 hours ago   142MB
```
#### Отправляю на Docker-HUB
```
% docker push constantinelipatkin/frontend:13.1
The push refers to repository [docker.io/constantinelipatkin/frontend]
e2ecd25639c1: Layer already exists 
bcb7ae696b7a: Layer already exists 
7a68866d30a9: Layer already exists 
5f70bf18a086: Layer already exists 
8f8614974cd6: Layer already exists 
ea4bc0cd4a93: Layer already exists 
fac199a5a1a5: Layer already exists 
5c77d760e1f4: Layer already exists 
33cf1b723f65: Layer already exists 
ea207a4854e7: Layer already exists 
608f3a074261: Layer already exists 
13.1: digest: sha256:b4a53c446d1a6b3959e171b0f40b8885c8521e32757ddc20d73933ba488f3252 size: 2607
%
% docker push constantinelipatkin/backend:13.1 
The push refers to repository [docker.io/constantinelipatkin/backend]
ffed9d1f13d8: Pushed 
db8e7736ad9e: Pushed 
a5966831f642: Pushed 
d1eeb5662d1b: Pushed 
5f70bf18a086: Mounted from constantinelipatkin/frontend 
99605b51636a: Pushed 
4c9efc7d50b2: Mounted from library/python 
5fffd512537b: Mounted from library/python 
fef7b0285d08: Mounted from library/python 
dad95716bb70: Mounted from library/python 
627d03f17169: Mounted from library/python 
6b9b07bf46f5: Mounted from library/python 
88139fe969ab: Mounted from library/python 
83f556e4c108: Mounted from library/python 
7362f7f77851: Mounted from library/python 
13.1: digest: sha256:9e9d9b710051ebeefae5ac00b13b6bc8df32a0882abc2aaaa99cb798f9426bd6 size: 3469
```
---
---

## Задание 1: подготовить тестовый конфиг для запуска приложения
Для начала следует подготовить запуск приложения в stage окружении с простыми настройками. Требования:
* под содержит в себе 2 контейнера — фронтенд, бекенд;
* регулируется с помощью deployment фронтенд и бекенд;
* база данных — через statefulset.

---
#### Создаю пространство имен "stage"
```
root@k8s-01:~# kubectl create namespace stage
namespace/stage created
```
#### Применяю конфигурацию
```
cat <<EOF | kubectl apply -f - -n stage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp
  name: frontend-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: constantinelipatkin/frontend:13.1
        name: frontend
        ports:
        - containerPort: 80
      - image: constantinelipatkin/backend:13.1
        name: backend
        ports:
        - containerPort: 9000
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql-db
spec:
  serviceName: “postgresql-db”
  selector:
    matchLabels:
      app: postgresql-db
  replicas: 1
  template:
    metadata:
      labels:
        app: postgresql-db
    spec:
      containers:
      - name: postgresql-db
        image: postgres:13-alpine
        ports:
        - containerPort: 5432
        env:
          - name: POSTGRES_DB
            value: news
          - name: POSTGRES_PASSWORD
            value: postgres
          - name: POSTGRES_USER
            value: postgres
---
apiVersion: v1
kind: Service
metadata:
  name: postgresql-db
spec:
  selector:
    app: postgresql-db
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f - -n stage
> ---
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   labels:
>     app: myapp
>   name: frontend-backend
> spec:
>   replicas: 1
>   selector:
>     matchLabels:
>       app: myapp
>   template:
>     metadata:
>       labels:
>         app: myapp
>     spec:
>       containers:
>       - image: constantinelipatkin/frontend:13.1
>         name: frontend
>         ports:
>         - containerPort: 80
>       - image: constantinelipatkin/backend:13.1
>         name: backend
>         ports:
>         - containerPort: 9000
> ---
> apiVersion: apps/v1
> kind: StatefulSet
> metadata:
>   name: postgresql-db
> spec:
>   serviceName: “postgresql-db”
>   selector:
>     matchLabels:
>       app: postgresql-db
>   replicas: 1
>   template:
>     metadata:
>       labels:
>         app: postgresql-db
>     spec:
>       containers:
>       - name: postgresql-db
>         image: postgres:13-alpine
>         ports:
>         - containerPort: 5432
>         env:
>           - name: POSTGRES_DB
>             value: news
>           - name: POSTGRES_PASSWORD
>             value: postgres
>           - name: POSTGRES_USER
>             value: postgres
> ---
> apiVersion: v1
> kind: Service
> metadata:
>   name: postgresql-db
> spec:
>   selector:
>     app: postgresql-db
>   ports:
>     - protocol: TCP
>       port: 5432
>       targetPort: 5432
> EOF
deployment.apps/frontend-backend created
statefulset.apps/postgresql-db created
service/postgresql-db created
```
#### Смотрю, что вышло
```
root@k8s-01:~# kubectl get deployments.apps -n stage 
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
frontend-backend   1/1     1            1           7m42s
root@k8s-01:~# 
root@k8s-01:~# kubectl get statefulsets.apps -n stage 
NAME            READY   AGE
postgresql-db   1/1     8m2s
root@k8s-01:~# 
root@k8s-01:~# kubectl get pods -n stage 
NAME                               READY   STATUS    RESTARTS   AGE
frontend-backend-8f87f56df-xkgn5   2/2     Running   0          8m40s
root@k8s-01:~# 
root@k8s-01:~#kubectl get service -n stage 
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
postgresql-db   ClusterIP   10.233.54.47   <none>        5432/TCP   9m5s
```
---
---

## Задание 2: подготовить конфиг для production окружения
Следующим шагом будет запуск приложения в production окружении. Требования сложнее:
* каждый компонент (база, бекенд, фронтенд) запускаются в своем поде, регулируются отдельными deployment’ами;
* для связи используются service (у каждого компонента свой);
* в окружении фронта прописан адрес сервиса бекенда;
* в окружении бекенда прописан адрес сервиса базы данных.

---
#### Применяю конфигурации:

#### - Front
```
cat <<EOF | kubectl apply -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp-front
  name: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-front
  template:
    metadata:
      labels:
        app: myapp-front
    spec:
      containers:
        - image: constantinelipatkin/frontend:13.1
          name: frontend
          ports:
            - containerPort: 8000
          env:
            - name: BASE_URL
              value: http://backend:9000
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: myapp-front
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8000
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f -
> ---
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   labels:
>     app: myapp-front
>   name: frontend
> spec:
>   replicas: 1
>   selector:
>     matchLabels:
>       app: myapp-front
>   template:
>     metadata:
>       labels:
>         app: myapp-front
>     spec:
>       containers:
>         - image: constantinelipatkin/frontend:13.1
>           name: frontend
>           ports:
>             - containerPort: 8000
>           env:
>             - name: BASE_URL
>               value: http://backend:9000
> ---
> apiVersion: v1
> kind: Service
> metadata:
>   name: frontend
> spec:
>   selector:
>     app: myapp-front
>   ports:
>     - protocol: TCP
>       port: 8080
>       targetPort: 8000
> EOF
deployment.apps/frontend created
service/frontend created
```
#### - Prod
```
cat <<EOF | kubectl apply -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp-back
  name: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-back
  template:
    metadata:
      labels:
        app: myapp-back
    spec:
      containers:
      - image: constantinelipatkin/backend:13.1
        name: backend 
        ports: 
          - containerPort: 9000
        env:
        - name: DATABASE_URL
          value: postgres://postgres:postgres@postgresql:5432/news
---
apiVersion: v1
kind: Service
metadata:
    name: backend
spec:
  selector:
    app: myapp-back
  ports:
    - protocol: TCP
      port: 9000
      targetPort: 9000
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f -
> ---
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   labels:
>     app: myapp-back
>   name: backend
> spec:
>   replicas: 1
>   selector:
>     matchLabels:
>       app: myapp-back
>   template:
>     metadata:
>       labels:
>         app: myapp-back
>     spec:
>       containers:
>       - image: constantinelipatkin/backend:13.1
>         name: backend 
>         ports: 
>           - containerPort: 9000
>         env:
>         - name: DATABASE_URL
>           value: postgres://postgres:postgres@postgresql:5432/news
> ---
> apiVersion: v1
> kind: Service
> metadata:
>     name: backend
> spec:
>   selector:
>     app: myapp-back
>   ports:
>     - protocol: TCP
>       port: 9000
>       targetPort: 9000
> EOF
deployment.apps/backend created
service/backend created
```
#### - DB
```
cat <<EOF | kubectl apply -f -
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgresql
spec:
  selector:
    matchLabels:
      app: myapp-db
  replicas: 1
  template:
    metadata:
      labels:
        app: myapp-db
    spec:
      containers:
      - name: postgresql
        image: postgres:13-alpine
        ports:
          - containerPort: 5432
        env:
          - name: POSTGRES_DB
            value: news
          - name: POSTGRES_PASSWORD
            value: postgres
          - name: POSTGRES_USER
            value: postgres
---
apiVersion: v1
kind: Service
metadata:
  name: postgresql
spec:
  selector:
    app: myapp-db
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f -
> ---
> apiVersion: apps/v1
> kind: Deployment
> metadata:
>   name: postgresql
> spec:
>   selector:
>     matchLabels:
>       app: myapp-db
>   replicas: 1
>   template:
>     metadata:
>       labels:
>         app: myapp-db
>     spec:
>       containers:
>       - name: postgresql
>         image: postgres:13-alpine
>         ports:
>           - containerPort: 5432
>         env:
>           - name: POSTGRES_DB
>             value: news
>           - name: POSTGRES_PASSWORD
>             value: postgres
>           - name: POSTGRES_USER
>             value: postgres
> ---
> apiVersion: v1
> kind: Service
> metadata:
>   name: postgresql
> spec:
>   selector:
>     app: myapp-db
>   ports:
>     - protocol: TCP
>       port: 5432
>       targetPort: 5432
> EOF
deployment.apps/postgresql created
service/postgresql created
```
#### Смотрю, что вышло
```
root@k8s-01:~# kubectl get all -o wide
NAME                                      READY   STATUS    RESTARTS      AGE     IP             NODE     NOMINATED NODE   READINESS GATES
pod/backend-6446b4dd4f-tpj75              1/1     Running   0             3m18s   10.233.87.53   k8s-03   <none>           <none>
pod/frontend-c65db47f4-8mjgp              1/1     Running   0             8m10s   10.233.77.48   k8s-05   <none>           <none>
pod/nfs-server-nfs-server-provisioner-0   1/1     Running   1 (10m ago)   16h     10.233.73.47   k8s-04   <none>           <none>
pod/postgresql-7b95c45bf9-4l46j           1/1     Running   0             2m4s    10.233.82.46   k8s-02   <none>           <none>

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/backend      ClusterIP   10.233.58.203   <none>        9000/TCP   3m18s   app=myapp-back
service/frontend     ClusterIP   10.233.44.253   <none>        8080/TCP   8m10s   app=myapp-front
service/kubernetes   ClusterIP   10.233.0.1      <none>        443/TCP    10m     <none>
service/postgresql   ClusterIP   10.233.30.96    <none>        5432/TCP   2m4s    app=myapp-db

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                              SELECTOR
deployment.apps/backend      1/1     1            1           3m18s   backend      constantinelipatkin/backend:13.1    app=myapp-back
deployment.apps/frontend     1/1     1            1           8m10s   frontend     constantinelipatkin/frontend:13.1   app=myapp-front
deployment.apps/postgresql   1/1     1            1           2m4s    postgresql   postgres:13-alpine                  app=myapp-db

NAME                                    DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES                              SELECTOR
replicaset.apps/backend-6446b4dd4f      1         1         1       3m18s   backend      constantinelipatkin/backend:13.1    app=myapp-back,pod-template-hash=6446b4dd4f
replicaset.apps/frontend-c65db47f4      1         1         1       8m10s   frontend     constantinelipatkin/frontend:13.1   app=myapp-front,pod-template-hash=c65db47f4
replicaset.apps/postgresql-7b95c45bf9   1         1         1       2m4s    postgresql   postgres:13-alpine                  app=myapp-db,pod-template-hash=7b95c45bf9

NAME                                                 READY   AGE   CONTAINERS               IMAGES
statefulset.apps/nfs-server-nfs-server-provisioner   1/1     21h   nfs-server-provisioner   quay.io/kubernetes_incubator/nfs-provisioner:v2.3.0
```
```
root@k8s-01:~# kubectl get deployments.apps 
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
backend      1/1     1            1           5m14s
frontend     1/1     1            1           10m
postgresql   1/1     1            1           4m
root@k8s-01:~#
root@k8s-01:~# kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend      ClusterIP   10.233.58.203   <none>        9000/TCP   5m46s
frontend     ClusterIP   10.233.44.253   <none>        8080/TCP   10m
kubernetes   ClusterIP   10.233.0.1      <none>        443/TCP    12m
postgresql   ClusterIP   10.233.30.96    <none>        5432/TCP   4m32s
root@k8s-01:~# 
root@k8s-01:~# kubectl get pods
NAME                                  READY   STATUS    RESTARTS      AGE
backend-6446b4dd4f-tpj75              1/1     Running   0             6m2s
frontend-c65db47f4-8mjgp              1/1     Running   0             10m
nfs-server-nfs-server-provisioner-0   1/1     Running   1 (12m ago)   16h
postgresql-7b95c45bf9-4l46j           1/1     Running   0             4m48s
```
---
---

## Задание 3 (*): добавить endpoint на внешний ресурс api
Приложению потребовалось внешнее api, и для его использования лучше добавить endpoint в кластер, направленный на это api. Требования:
* добавлен endpoint до внешнего api (например, геокодер).

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

В качестве решения прикрепите к ДЗ конфиг файлы для деплоя. Прикрепите скриншоты вывода команды kubectl со списком запущенных объектов каждого типа (pods, deployments, statefulset, service) или скриншот из самого Kubernetes, что сервисы подняты и работают.

---
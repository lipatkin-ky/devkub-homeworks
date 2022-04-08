# Домашнее задание к занятию "12.2 Команды для работы с Kubernetes"
Кластер — это сложная система, с которой крайне редко работает один человек. Квалифицированный devops умеет наладить работу всей команды, занимающейся каким-либо сервисом.
После знакомства с кластером вас попросили выдать доступ нескольким разработчикам. Помимо этого требуется служебный аккаунт для просмотра логов.

---
---
## Задание 1: Запуск пода из образа в деплойменте
Для начала следует разобраться с прямым запуском приложений из консоли. Такой подход поможет быстро развернуть инструменты отладки в кластере. Требуется запустить деплоймент на основе образа из hello world уже через deployment. Сразу стоит запустить 2 копии приложения (replicas=2). 

Требования:
 * пример из hello world запущен в качестве deployment
 * количество реплик в deployment установлено в 2
 * наличие deployment можно проверить командой kubectl get deployment
 * наличие подов можно проверить командой kubectl get pods
---
```
root@k8s-01:~# kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
deployment.apps/nginx-deployment created
```
```
root@k8s-01:~# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     3            0           8s
```
```
root@k8s-01:~# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           16s
```
```
root@k8s-01:~# kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9456bbbf9-7drqx   1/1     Running   0          3m19s
nginx-deployment-9456bbbf9-dnwn4   1/1     Running   0          3m19s
nginx-deployment-9456bbbf9-kfwlp   1/1     Running   0          3m19s
```
```
root@k8s-01:~# kubectl edit deployments.apps
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"
nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  creationTimestamp: "2022-03-23T19:37:08Z"
  generation: 1
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
  resourceVersion: "16462"
  uid: 0d42b585-f1a6-423b-8262-7e37268bcfae
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
"/tmp/kubectl-edit-1283599483.yaml" 71L, 2295C written                                                                                                                              
deployment.apps/nginx-deployment edited
```
```
root@k8s-01:~# kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9456bbbf9-7drqx   1/1     Running   0          13m
nginx-deployment-9456bbbf9-kfwlp   1/1     Running   0          13m
```  
```
root@k8s-01:~# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2/2     2            2           13m
```
---
---
## Задание 2: Просмотр логов для разработки
Разработчикам крайне важно получать обратную связь от штатно работающего приложения и, еще важнее, об ошибках в его работе. 
Требуется создать пользователя и выдать ему доступ на чтение конфигурации и логов подов в app-namespace.

Требования: 
 * создан новый токен доступа для пользователя
 * пользователь прописан в локальный конфиг (~/.kube/config, блок users)
 * пользователь может просматривать логи подов и их конфигурацию (kubectl logs pod <pod_id>, kubectl describe pod <pod_id>)
---
#### 1. Создаю app-namespace
```
root@k8s-01:~# kubectl create namespace app-namespace
namespace/app-namespace created
```
```
root@k8s-01:~# kubectl get namespaces
NAME              STATUS   AGE
app-namespace     Active   3s
default           Active   7d18h
kube-node-lease   Active   7d18h
kube-public       Active   7d18h
kube-system       Active   7d18h
```
#### 2. Создаю УЗ службы в app-namespace
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: test
  namespace: app-namespace
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f -
> apiVersion: v1
> kind: ServiceAccount
> metadata:
>   name: test
>   namespace: app-namespace
> EOF


serviceaccount/test created
```
```
root@k8s-01:~# kubectl get serviceaccounts -n app-namespace
NAME      SECRETS   AGE
default   1         57m
test      1         56m
```
#### 3. Создаю роль
```
cat <<EOF | kubectl apply -f -
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: viewer
  namespace: app-namespace
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", “logs”, "describe"]
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f -
> kind: Role
> apiVersion: rbac.authorization.k8s.io/v1
> metadata:
>   name: viewer
>   namespace: app-namespace
> rules:
> - apiGroups: [""]
>   resources: ["pods"]
>   verbs: ["get", "list", "watch", “logs”, "describe"]
> EOF

role.rbac.authorization.k8s.io/viewer created
```
```
root@k8s-01:~# kubectl get roles -n app-namespace
NAME                      CREATED AT
admin                     2022-03-31T12:38:24Z
viewer                    2022-03-31T12:54:36Z
```
#### 4. Привязываю роль к пользователю
```
cat <<EOF | kubectl apply -f -
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: role-viewer
  namespace: app-namespace
subjects:
- kind: ServiceAccount
  name: test
  namespace: app-namespace
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: view
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f -
> kind: RoleBinding
> apiVersion: rbac.authorization.k8s.io/v1
> metadata:
>   name: role-viewer
>   namespace: app-namespace
> subjects:
> - kind: ServiceAccount
>   name: test
>   namespace: app-namespace
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: Role
>   name: view
> EOF

rolebinding.rbac.authorization.k8s.io/role-viewer created
```
```
root@k8s-01:~# kubectl get rolebindings --namespace app-namespace
NAME          ROLE        AGE
role-viewer   Role/view   7s
```






---
---
## Задание 3: Изменение количества реплик 
Поработав с приложением, вы получили запрос на увеличение количества реплик приложения для нагрузки. Необходимо изменить запущенный deployment, увеличив количество реплик до 5. Посмотрите статус запущенных подов после увеличения реплик. 

Требования:
 * в deployment из задания 1 изменено количество реплик на 5
 * проверить что все поды перешли в статус running (kubectl get pods)
---
```
root@k8s-01:~# kubectl edit deployments.apps
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app":"nginx"},"name":"nginx-deployment","namespace":"default"},"spec":{"replicas":3,"selector":{"matchLabels":{"app":"
nginx"}},"template":{"metadata":{"labels":{"app":"nginx"}},"spec":{"containers":[{"image":"nginx:1.14.2","name":"nginx","ports":[{"containerPort":80}]}]}}}}
  creationTimestamp: "2022-03-23T19:37:08Z"
  generation: 2
  labels:
    app: nginx
  name: nginx-deployment
  namespace: default
  resourceVersion: "18065"
  uid: 0d42b585-f1a6-423b-8262-7e37268bcfae
spec:
  progressDeadlineSeconds: 600
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.14.2
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
"/tmp/kubectl-edit-637963326.yaml" 71L, 2295C written                                                                                                                               
deployment.apps/nginx-deployment edited
```
```
root@k8s-01:~# kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5/5     5            5           48m
```
```
root@k8s-01:~# kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-9456bbbf9-7drqx   1/1     Running   0          48m
nginx-deployment-9456bbbf9-djjr7   1/1     Running   0          16s
nginx-deployment-9456bbbf9-f7lfr   1/1     Running   0          16s
nginx-deployment-9456bbbf9-kfwlp   1/1     Running   0          48m
nginx-deployment-9456bbbf9-vfqvq   1/1     Running   0          16s
```
---
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

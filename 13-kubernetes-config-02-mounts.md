# Домашнее задание к занятию "13.2 разделы и монтирование"
Приложение запущено и работает, но время от времени появляется необходимость передавать между бекендами данные. А сам бекенд генерирует статику для фронта. Нужно оптимизировать это.
Для настройки NFS сервера можно воспользоваться следующей инструкцией (производить под пользователем на сервере, у которого есть доступ до kubectl):
* установить helm: curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
* добавить репозиторий чартов: helm repo add stable https://charts.helm.sh/stable && helm repo update
* установить nfs-server через helm: helm install nfs-server stable/nfs-server-provisioner

В конце установки будет выдан пример создания PVC для этого сервера.

---
```
root@k8s-01:~# curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 11156  100 11156    0     0  25645      0 --:--:-- --:--:-- --:--:-- 25587
Downloading https://get.helm.sh/helm-v3.8.1-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
```
```
root@k8s-01:~# helm repo add stable https://charts.helm.sh/stable && helm repo update
"stable" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```
```
root@k8s-01:~# helm install nfs-server stable/nfs-server-provisioner
WARNING: This chart is deprecated
NAME: nfs-server
LAST DEPLOYED: Fri Apr  8 13:04:52 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The NFS Provisioner service has now been installed.

A storage class named 'nfs' has now been created
and is available to provision dynamic volumes.

You can use this storageclass by creating a `PersistentVolumeClaim` with the
correct storageClassName attribute. For example:

    ---
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: test-dynamic-volume-claim
    spec:
      storageClassName: "nfs"
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 100Mi
```
---
---

## Задание 1: подключить для тестового конфига общую папку
В stage окружении часто возникает необходимость отдавать статику бекенда сразу фронтом. Проще всего сделать это через общую папку. Требования:
* в поде подключена общая папка между контейнерами (например, /static);
* после записи чего-либо в контейнере с беком файлы можно получить из контейнера с фронтом.

---
#### Запускаю Pod в следующей конфигурации
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-int-volumes
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: "/static"
          name: static
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      volumeMounts:
        - mountPath: "/tmp/cache"
          name: static
  volumes:
    - name: static
      emptyDir: {}
EOF
```
```
root@k8s-01:~# cat <<EOF | kubectl apply -f -
> apiVersion: v1
> kind: Pod
> metadata:
>   name: pod-int-volumes
> spec:
>   containers:
>     - name: nginx
>       image: nginx
>       volumeMounts:
>         - mountPath: "/static"
>           name: static
>     - name: busybox
>       image: busybox
>       command: ["sleep", "3600"]
>       volumeMounts:
>         - mountPath: "/tmp/cache"
>           name: static
>   volumes:
>     - name: static
>       emptyDir: {}
> EOF
pod/pod-int-volumes created
```
```
root@k8s-01:~# kubectl get pods
NAME                                  READY   STATUS    RESTARTS       AGE
multitool-587cc865-wqdlv              2/2     Running   2 (56m ago)    2d21h
multitool-587cc865-xs5dx              2/2     Running   2 (2d8h ago)   2d21h
multitool-587cc865-zrk7q              2/2     Running   2 (56m ago)    2d21h
nfs-server-nfs-server-provisioner-0   1/1     Running   0              53m
pod-int-volumes                       2/2     Running   0              43s
```
#### Проверяю наличие файлов
```
root@k8s-01:~# kubectl exec pod-int-volumes -c busybox -- ls -la /tmp/cache
total 8
drwxrwxrwx    2 root     root          4096 Apr  8 14:08 .
drwxrwxrwt    1 root     root          4096 Apr  8 13:57 ..
root@k8s-01:~#
root@k8s-01:~# kubectl exec pod-int-volumes -c nginx -- ls -la /static
total 8
drwxrwxrwx 2 root root 4096 Apr  8 14:08 .
drwxr-xr-x 1 root root 4096 Apr  8 13:57 ..
```
#### Записываю в контейнер "busybox" и проверяю наличие файлов
```
root@k8s-01:~# kubectl exec pod-int-volumes -c busybox -- sh -c 'echo "test" > /tmp/cache/test.txt'
root@k8s-01:~#
root@k8s-01:~# kubectl exec pod-int-volumes -c busybox -- ls -la /tmp/cache
total 12
drwxrwxrwx    2 root     root          4096 Apr  8 14:10 .
drwxrwxrwt    1 root     root          4096 Apr  8 13:57 ..
-rw-r--r--    1 root     root             5 Apr  8 14:10 test.txt
root@k8s-01:~# 
root@k8s-01:~# kubectl exec pod-int-volumes -c nginx -- ls -la /static
total 12
drwxrwxrwx 2 root root 4096 Apr  8 14:10 .
drwxr-xr-x 1 root root 4096 Apr  8 13:57 ..
-rw-r--r-- 1 root root    5 Apr  8 14:10 test.txt
```
#### Читаю в контейнере "nginx"
```
root@k8s-01:~# kubectl exec pod-int-volumes -c nginx -- sh -c 'cat /static/test.txt'
test
```
---
---
## Задание 2: подключить общую папку для прода
Поработав на stage, доработки нужно отправить на прод. В продуктиве у нас контейнеры крутятся в разных подах, поэтому потребуется PV и связь через PVC. Сам PV должен быть связан с NFS сервером. Требования:
* все бекенды подключаются к одному PV в режиме ReadWriteMany;
* фронтенды тоже подключаются к этому же PV с таким же режимом;
* файлы, созданные бекендом, должны быть доступны фронту.

---
#### Установка дополнительного пакета на всех нодах кластера
```
$ sudo apt-get install -y nfs-common
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  keyutils libevent-2.1-7 libnfsidmap2 libtirpc-common libtirpc3 rpcbind
Suggested packages:
  open-iscsi watchdog
The following NEW packages will be installed:
  keyutils libevent-2.1-7 libnfsidmap2 libtirpc-common libtirpc3 nfs-common rpcbind
0 upgraded, 7 newly installed, 0 to remove and 61 not upgraded.
Need to get 542 kB of archives.
After this operation, 1,922 kB of additional disk space will be used.
Get:1 http://mirror.yandex.ru/ubuntu focal/main amd64 libtirpc-common all 1.2.5-1 [7,632 B]
Get:2 http://mirror.yandex.ru/ubuntu focal/main amd64 libtirpc3 amd64 1.2.5-1 [77.2 kB]
Get:3 http://mirror.yandex.ru/ubuntu focal/main amd64 rpcbind amd64 1.2.5-8 [42.8 kB]
Get:4 http://mirror.yandex.ru/ubuntu focal/main amd64 keyutils amd64 1.6-6ubuntu1 [45.0 kB]
Get:5 http://mirror.yandex.ru/ubuntu focal/main amd64 libevent-2.1-7 amd64 2.1.11-stable-1 [138 kB]
Get:6 http://mirror.yandex.ru/ubuntu focal/main amd64 libnfsidmap2 amd64 0.25-5.1ubuntu1 [27.9 kB]
Get:7 http://mirror.yandex.ru/ubuntu focal-updates/main amd64 nfs-common amd64 1:1.3.4-2.5ubuntu3.4 [204 kB]
Fetched 542 kB in 0s (11.0 MB/s)         
Selecting previously unselected package libtirpc-common.
(Reading database ... 102434 files and directories currently installed.)
Preparing to unpack .../0-libtirpc-common_1.2.5-1_all.deb ...
Unpacking libtirpc-common (1.2.5-1) ...
Selecting previously unselected package libtirpc3:amd64.
Preparing to unpack .../1-libtirpc3_1.2.5-1_amd64.deb ...
Unpacking libtirpc3:amd64 (1.2.5-1) ...
Selecting previously unselected package rpcbind.
Preparing to unpack .../2-rpcbind_1.2.5-8_amd64.deb ...
Unpacking rpcbind (1.2.5-8) ...
Selecting previously unselected package keyutils.
Preparing to unpack .../3-keyutils_1.6-6ubuntu1_amd64.deb ...
Unpacking keyutils (1.6-6ubuntu1) ...
Selecting previously unselected package libevent-2.1-7:amd64.
Preparing to unpack .../4-libevent-2.1-7_2.1.11-stable-1_amd64.deb ...
Unpacking libevent-2.1-7:amd64 (2.1.11-stable-1) ...
Selecting previously unselected package libnfsidmap2:amd64.
Preparing to unpack .../5-libnfsidmap2_0.25-5.1ubuntu1_amd64.deb ...
Unpacking libnfsidmap2:amd64 (0.25-5.1ubuntu1) ...
Selecting previously unselected package nfs-common.
Preparing to unpack .../6-nfs-common_1%3a1.3.4-2.5ubuntu3.4_amd64.deb ...
Unpacking nfs-common (1:1.3.4-2.5ubuntu3.4) ...
Setting up libtirpc-common (1.2.5-1) ...
Setting up libevent-2.1-7:amd64 (2.1.11-stable-1) ...
Setting up keyutils (1.6-6ubuntu1) ...
Setting up libnfsidmap2:amd64 (0.25-5.1ubuntu1) ...
Setting up libtirpc3:amd64 (1.2.5-1) ...
Setting up rpcbind (1.2.5-8) ...
Created symlink /etc/systemd/system/multi-user.target.wants/rpcbind.service → /lib/systemd/system/rpcbind.service.
Created symlink /etc/systemd/system/sockets.target.wants/rpcbind.socket → /lib/systemd/system/rpcbind.socket.
Setting up nfs-common (1:1.3.4-2.5ubuntu3.4) ...

Creating config file /etc/idmapd.conf with new version
Adding system user `statd' (UID 110) ...
Adding new user `statd' (UID 110) with group `nogroup' ...
Not creating home directory `/var/lib/nfs'.
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
Created symlink /etc/systemd/system/remote-fs.target.wants/nfs-client.target → /lib/systemd/system/nfs-client.target.
nfs-utils.service is a disabled or a static unit, not starting it.
Processing triggers for systemd (245.4-4ubuntu3.15) ...
Processing triggers for man-db (2.9.1-1) ...
Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
```
#### Добалвяю/обновляю конфигурацию
#### - PV
```
cat <<EOF | kubectl apply -f -
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-static
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  capacity:
       storage: 1Gi
  hostPath:
    path: /tmp/pv
EOF
```
#### - PVC
```
cat <<EOF | kubectl apply -f -
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc
spec:
  storageClassName: nfs
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
EOF
```
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
          volumeMounts:
            - mountPath: /static
              name: pv
      volumes: 
        - name: pv
          persistentVolumeClaim:
            claimName: pvc
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
#### - Back
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
          value: postgres://postgres:postgres@db:5432/news
        volumeMounts:
          - mountPath: /static
            name: pv
      volumes:
        - name: pv
          persistentVolumeClaim:
            claimName: pvc
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
#### Смотрю Pod-ы
```
root@k8s-01:~# kubectl get pods
NAME                                  READY   STATUS    RESTARTS       AGE
backend-6847d9c8dd-cltv4              1/1     Running   0              20m
frontend-869dc8cff5-j8c9s             1/1     Running   0              20m
nfs-server-nfs-server-provisioner-0   1/1     Running   1 (110m ago)   18h
```
#### Создаю "Testfile" в Pod-е "backend"
```
root@k8s-01:~#  kubectl exec -it backend-6847d9c8dd-cltv4 -- bash
root@backend-6847d9c8dd-cltv4:/app#
root@backend-6847d9c8dd-cltv4:/app# touch /static/Testfile
root@backend-6847d9c8dd-cltv4:/# cd /static/
root@backend-6847d9c8dd-cltv4:/static# ls
Testfile
root@backend-6847d9c8dd-cltv4:/static#
root@backend-6847d9c8dd-cltv4:/static# exit
```
#### Проверяю наличие "Testfile" в Pod-е "frontend"
```
root@k8s-01:~# kubectl exec -it frontend-869dc8cff5-j8c9s -- bash
root@frontend-869dc8cff5-j8c9s:/app# 
root@frontend-869dc8cff5-j8c9s:/app# ls -l /static/
total 0
-rw-r--r-- 1 root root 0 Apr  9 12:01 Testfile
root@frontend-869dc8cff5-j8c9s:/app# exit
```
---
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
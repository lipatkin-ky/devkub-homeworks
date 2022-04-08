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

---
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
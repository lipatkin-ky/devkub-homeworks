# Домашнее задание к занятию "13.3 работа с kubectl"

## Задание 1: проверить работоспособность каждого компонента
Для проверки работы можно использовать 2 способа: port-forward и exec. Используя оба способа, проверьте каждый компонент:
* сделайте запросы к бекенду;
* сделайте запросы к фронту;
* подключитесь к базе данных.

---

---
---

## Задание 2: ручное масштабирование

При работе с приложением иногда может потребоваться вручную добавить пару копий. Используя команду kubectl scale, попробуйте увеличить количество бекенда и фронта до 3. Проверьте, на каких нодах оказались копии после каждого действия (kubectl describe, kubectl get pods -o wide). После уменьшите количество копий до 1.

---
```
root@k8s-01:~# kubectl get nodes 
NAME     STATUS   ROLES                  AGE   VERSION
k8s-01   Ready    control-plane,master   21d   v1.23.5
k8s-02   Ready    <none>                 21d   v1.23.5
k8s-03   Ready    <none>                 21d   v1.23.5
k8s-04   Ready    <none>                 21d   v1.23.5
k8s-05   Ready    <none>                 21d   v1.23.5
```
```
root@k8s-01:~# kubectl get pods -o wide 
NAME                                  READY   STATUS    RESTARTS      AGE     IP             NODE     NOMINATED NODE   READINESS GATES
backend-6847d9c8dd-nrscn              1/1     Running   0             41m     10.233.87.60   k8s-03   <none>           <none>
frontend-869dc8cff5-qsj8k             1/1     Running   0             7m49s   10.233.77.77   k8s-05   <none>           <none>
nfs-server-nfs-server-provisioner-0   1/1     Running   2 (62m ago)   5d2h    10.233.73.48   k8s-04   <none>           <none>
postgresql-7b95c45bf9-nzzcz           1/1     Running   0             54m     10.233.87.58   k8s-03   <none>           <none>
root@k8s-01:~# 
```
```
root@k8s-01:~# kubectl scale --replicas=3 deployment backend frontend 
deployment.apps/backend scaled
deployment.apps/frontend scaled
```
```
root@k8s-01:~# kubectl get pods -o wide 
NAME                                  READY   STATUS    RESTARTS      AGE    IP             NODE     NOMINATED NODE   READINESS GATES
backend-6847d9c8dd-hnzdm              1/1     Running   0             61s    10.233.77.80   k8s-05   <none>           <none>
backend-6847d9c8dd-nrscn              1/1     Running   0             42m    10.233.87.60   k8s-03   <none>           <none>
backend-6847d9c8dd-wt7f7              1/1     Running   0             61s    10.233.82.74   k8s-02   <none>           <none>
frontend-869dc8cff5-5n7qp             1/1     Running   0             61s    10.233.82.75   k8s-02   <none>           <none>
frontend-869dc8cff5-dhpsp             1/1     Running   0             61s    10.233.73.69   k8s-04   <none>           <none>
frontend-869dc8cff5-qsj8k             1/1     Running   0             9m8s   10.233.77.77   k8s-05   <none>           <none>
nfs-server-nfs-server-provisioner-0   1/1     Running   2 (63m ago)   5d2h   10.233.73.48   k8s-04   <none>           <none>
postgresql-7b95c45bf9-nzzcz           1/1     Running   0             55m    10.233.87.58   k8s-03   <none>           <none>
root@k8s-01:~# 
```
```
root@k8s-01:~# kubectl scale --replicas=1 deployment backend frontend 
deployment.apps/backend scaled
deployment.apps/frontend scaled
```
```
root@k8s-01:~# kubectl get pods -o wide 
NAME                                  READY   STATUS    RESTARTS      AGE    IP             NODE     NOMINATED NODE   READINESS GATES
backend-6847d9c8dd-nrscn              1/1     Running   0             43m    10.233.87.60   k8s-03   <none>           <none>
frontend-869dc8cff5-qsj8k             1/1     Running   0             10m    10.233.77.77   k8s-05   <none>           <none>
nfs-server-nfs-server-provisioner-0   1/1     Running   2 (64m ago)   5d2h   10.233.73.48   k8s-04   <none>           <none>
postgresql-7b95c45bf9-nzzcz           1/1     Running   0             56m    10.233.87.58   k8s-03   <none>           <none>
root@k8s-01:~# 
```
```
root@k8s-01:~# kubectl describe pods backend-6847d9c8dd-nrscn frontend-869dc8cff5-qsj8k | grep k8s
Node:         k8s-03/10.129.0.13
  Normal  Scheduled  44m   default-scheduler  Successfully assigned default/backend-6847d9c8dd-nrscn to k8s-03
Node:         k8s-05/10.129.0.36
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/frontend-869dc8cff5-qsj8k to k8s-05
root@k8s-01:~# 
```
#### Месторасположение Pod-ов до и после изменения их копий не изменилось.

#### Проделал еще несколько итераций и получил следующее:
```
root@k8s-01:~# kubectl get pods -o wide 
NAME                                  READY   STATUS    RESTARTS      AGE     IP             NODE     NOMINATED NODE   READINESS GATES
backend-6847d9c8dd-hthph              1/1     Running   0             92s     10.233.77.88   k8s-05   <none>           <none>
frontend-869dc8cff5-jpt96             1/1     Running   0             2m25s   10.233.87.75   k8s-03   <none>           <none>
nfs-server-nfs-server-provisioner-0   1/1     Running   2 (68m ago)   5d2h    10.233.73.48   k8s-04   <none>           <none>
postgresql-7b95c45bf9-nzzcz           1/1     Running   0             60m     10.233.87.58   k8s-03   <none>           <none>
```
```
root@k8s-01:~# kubectl describe pods backend-6847d9c8dd-hthph frontend-869dc8cff5-jpt96 | grep k8s
Node:         k8s-05/10.129.0.36
  Normal  Scheduled  2m23s  default-scheduler  Successfully assigned default/backend-6847d9c8dd-hthph to k8s-05
Node:         k8s-03/10.129.0.13
  Normal  Scheduled  3m16s  default-scheduler  Successfully assigned default/frontend-869dc8cff5-jpt96 to k8s-03
```
#### Месторасположение Pod-ов до и после изменения их копий изменилось.

---
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
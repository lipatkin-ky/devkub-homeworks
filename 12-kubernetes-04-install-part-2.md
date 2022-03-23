# Домашнее задание к занятию "12.4 Развертывание кластера на собственных серверах, лекция 2"
Новые проекты пошли стабильным потоком. Каждый проект требует себе несколько кластеров: под тесты и продуктив. Делать все руками — не вариант, поэтому стоит автоматизировать подготовку новых кластеров.

---
---

## Задание 1: Подготовить инвентарь kubespray
Новые тестовые кластеры требуют типичных простых настроек. Нужно подготовить инвентарь и проверить его работу. Требования к инвентарю:
* подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды;
* в качестве CRI — containerd;
* запуск etcd производить на мастере.
---
```
 ~ % yc compute instance list   
+----------------------+--------+---------------+---------+----------------+-------------+
|          ID          |  NAME  |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP |
+----------------------+--------+---------------+---------+----------------+-------------+
| epd6vjd4g4244ag11h2f | k8s-05 | ru-central1-b | RUNNING | 51.250.104.248 | 10.129.0.36 |
| epdjf1sc2qql5en1bpk2 | k8s-03 | ru-central1-b | RUNNING | 51.250.104.11  | 10.129.0.13 |
| epdjgb3vkgb5j412v25d | k8s-04 | ru-central1-b | RUNNING | 51.250.100.254 | 10.129.0.10 |
| epdq0mmrs8f8qfckvvtg | k8s-02 | ru-central1-b | RUNNING | 51.250.102.30  | 10.129.0.22 |
| epdseclp0oav0msourf3 | k8s-01 | ru-central1-b | RUNNING | 51.250.104.217 | 10.129.0.12 |
+----------------------+--------+---------------+---------+----------------+-------------+
```
```
 kubespray % cat inventory/netology-new/inventory.ini                                 
# ## Configure 'ip' variable to bind kubernetes services on a
# ## different ip than the default iface
# ## We should set etcd_member_name for etcd cluster. The node that is not a etcd member do not need to set the value, or can set the empty string value.
[all]
k8s-01 ansible_host=51.250.104.217 ip=10.129.0.12 etcd_member_name=etcd1
k8s-02 ansible_host=51.250.102.30 ip=10.129.0.22 #etcd_member_name=etcd2
k8s-03 ansible_host=51.250.104.11 ip=10.129.0.13 #etcd_member_name=etcd3
k8s-04 ansible_host=51.250.100.254 ip=10.129.0.10 #etcd_member_name=etcd4
k8s-05 ansible_host=51.250.104.248 ip=10.129.0.36 #etcd_member_name=etcd5

# ## configure a bastion host if your nodes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
k8s-01

[etcd]
k8s-01

[kube_node]
k8s-02
k8s-03
k8s-04
k8s-05

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```
``` 
kubespray % ansible-playbook -i inventory/netology-new/inventory.ini cluster.yml -b -v
-//-
Wednesday 23 March 2022  20:40:36 +0300 (0:00:00.195)       0:20:48.848 ******* 

PLAY RECAP ************************************************************************************************************************************************************************************************************
k8s-01                     : ok=735  changed=149  unreachable=0    failed=0    skipped=1239 rescued=0    ignored=5   
k8s-02                     : ok=514  changed=97   unreachable=0    failed=0    skipped=728  rescued=0    ignored=1   
k8s-03                     : ok=514  changed=97   unreachable=0    failed=0    skipped=727  rescued=0    ignored=1   
k8s-04                     : ok=514  changed=97   unreachable=0    failed=0    skipped=727  rescued=0    ignored=1   
k8s-05                     : ok=514  changed=97   unreachable=0    failed=0    skipped=727  rescued=0    ignored=1   
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Wednesday 23 March 2022  20:40:36 +0300 (0:00:00.199)       0:20:49.048 ******* 
=============================================================================== 
kubernetes/preinstall : Install packages requirements --------------------------------------------------------------------------------------------------------------------------------------------------------- 56.16s
kubernetes/control-plane : kubeadm | Initialize first master -------------------------------------------------------------------------------------------------------------------------------------------------- 36.83s
kubernetes/kubeadm : Join to cluster -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 26.85s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------------------------- 19.15s
kubernetes/preinstall : Preinstall | wait for the apiserver to be running ------------------------------------------------------------------------------------------------------------------------------------- 16.71s
network_plugin/calico : Wait for calico kubeconfig to be created ---------------------------------------------------------------------------------------------------------------------------------------------- 16.09s
kubernetes/preinstall : Update package management cache (APT) ------------------------------------------------------------------------------------------------------------------------------------------------- 15.80s
kubernetes-apps/ansible : Kubernetes Apps | Start Resources --------------------------------------------------------------------------------------------------------------------------------------------------- 14.10s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------------------------- 13.52s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------------------------- 12.21s
kubernetes-apps/ansible : Kubernetes Apps | Lay Down CoreDNS templates ---------------------------------------------------------------------------------------------------------------------------------------- 12.17s
container-engine/containerd : containerd | Unpack containerd archive ------------------------------------------------------------------------------------------------------------------------------------------ 10.87s
download : download_container | Download image if required ---------------------------------------------------------------------------------------------------------------------------------------------------- 10.44s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------- 9.80s
etcd : reload etcd --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 9.61s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------- 8.44s
download : download_container | Download image if required ----------------------------------------------------------------------------------------------------------------------------------------------------- 8.41s
network_plugin/calico : Start Calico resources ----------------------------------------------------------------------------------------------------------------------------------------------------------------- 7.67s
network_plugin/calico : Calico | Create calico manifests ------------------------------------------------------------------------------------------------------------------------------------------------------- 7.63s
container-engine/crictl : extract_file | Unpacking archive ----------------------------------------------------------------------------------------------------------------------------------------------------- 7.37s
```
``` 
~ % ssh 51.250.104.217 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-96-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Last login: Wed Mar 23 17:40:33 2022 from 91.x.x.x
```
```
root@k8s-01:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.5", GitCommit:"c285e781331a3785a7f436042c65c5641ce8a9e9", GitTreeState:"clean", BuildDate:"2022-03-16T15:58:47Z", GoVersion:"go1.17.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.5", GitCommit:"c285e781331a3785a7f436042c65c5641ce8a9e9", GitTreeState:"clean", BuildDate:"2022-03-16T15:52:18Z", GoVersion:"go1.17.8", Compiler:"gc", Platform:"linux/amd64"}
```
```
@k8s-01:~$ sudo -i
```
```
@k8s-01:~# kubectl get nodes
NAME     STATUS   ROLES                  AGE     VERSION
k8s-01   Ready    control-plane,master   7m23s   v1.23.5
k8s-02   Ready    <none>                 5m54s   v1.23.5
k8s-03   Ready    <none>                 5m49s   v1.23.5
k8s-04   Ready    <none>                 5m49s   v1.23.5
k8s-05   Ready    <none>                 5m54s   v1.23.5
```
```
root@k8s-01:~# kubectl cluster-info 
Kubernetes control plane is running at https://127.0.0.1:6443

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
```
root@k8s-01:~# kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS        AGE
kube-system   calico-kube-controllers-75fcdd655b-jtm22   1/1     Running   0               4m45s
kube-system   calico-node-52jcs                          1/1     Running   0               5m44s
kube-system   calico-node-8ckqw                          1/1     Running   0               5m44s
kube-system   calico-node-fpt7c                          1/1     Running   0               5m44s
kube-system   calico-node-pfkdp                          1/1     Running   0               5m44s
kube-system   calico-node-z7ctg                          1/1     Running   0               5m44s
kube-system   coredns-76b4fb4578-78rxf                   1/1     Running   0               4m15s
kube-system   coredns-76b4fb4578-kqh8q                   1/1     Running   0               4m1s
kube-system   dns-autoscaler-7979fb6659-gqkxh            1/1     Running   0               4m7s
kube-system   kube-apiserver-k8s-01                      1/1     Running   1               7m52s
kube-system   kube-controller-manager-k8s-01             1/1     Running   2 (2m52s ago)   7m59s
kube-system   kube-proxy-4k64p                           1/1     Running   0               6m21s
kube-system   kube-proxy-lg9mh                           1/1     Running   0               6m21s
kube-system   kube-proxy-m9kp2                           1/1     Running   0               6m21s
kube-system   kube-proxy-v9thx                           1/1     Running   0               6m21s
kube-system   kube-proxy-wvl2s                           1/1     Running   0               6m21s
kube-system   kube-scheduler-k8s-01                      1/1     Running   2 (2m53s ago)   7m59s
kube-system   nginx-proxy-k8s-02                         1/1     Running   0               6m32s
kube-system   nginx-proxy-k8s-03                         1/1     Running   0               6m27s
kube-system   nginx-proxy-k8s-04                         1/1     Running   0               6m27s
kube-system   nginx-proxy-k8s-05                         1/1     Running   0               6m32s
kube-system   nodelocaldns-cx9kq                         1/1     Running   0               4m6s
kube-system   nodelocaldns-d62vz                         1/1     Running   0               4m6s
kube-system   nodelocaldns-dsvwm                         1/1     Running   0               4m6s
kube-system   nodelocaldns-nq8wt                         1/1     Running   0               4m6s
kube-system   nodelocaldns-xgph8                         1/1     Running   0               4m6s
```
---
---

## Задание 2 (*): подготовить и проверить инвентарь для кластера в AWS
Часть новых проектов хотят запускать на мощностях AWS. Требования похожи:
* разворачивать 5 нод: 1 мастер и 4 рабочие ноды;
* работать должны на минимально допустимых EC2 — t3.small.
---
Полагаю, что YC, что выше, вполне подойдет)))

---
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---

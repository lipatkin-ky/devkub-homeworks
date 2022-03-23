# Домашнее задание к занятию "12.3 Развертывание кластера на собственных серверах, лекция 1"
Поработав с персональным кластером, можно заняться проектами. Вам пришла задача подготовить кластер под новый проект.

---
---
## Задание 1: Описать требования к кластеру
Сначала проекту необходимо определить требуемые ресурсы. Известно, что проекту нужны база данных, система кеширования, а само приложение состоит из бекенда и фронтенда. Опишите, какие ресурсы нужны, если известно:

* база данных должна быть отказоустойчивой (не менее трех копий, master-slave) и потребляет около 4 ГБ ОЗУ в работе;
* кэш должен быть аналогично отказоустойчивый, более трех копий, потребление: 4 ГБ ОЗУ, 1 ядро;
* фронтенд обрабатывает внешние запросы быстро, отдавая статику: не более 50 МБ ОЗУ на каждый экземпляр;
* бекенду требуется больше: порядка 600 МБ ОЗУ и по 1 ядру на копию.

Требования: опишите, сколько нод в кластере и сколько ресурсов (ядра, ОЗУ, диск) нужно для запуска приложения. Расчет вести из необходимости запуска 5 копий фронтенда и 10 копий бекенда, база и кеш.

---

|         |HDD  |RAM  |CPU  |redundancy |count  |     |Sum HDD  |Sum RAM  |Sum CPU  |
|---      |---  |---  |---  |---        |---    |---  |---      |---      |---      |
|DB       |20   |4    |8    |1,5        |3      |     |90       |18       |36       |
|Front    |3    |0,05 |2    |1,5        |5      |     |22,5     |0,375    |15       |
|Back     |3    |0,6  |1    |1,5        |10     |     |45       |9        |15       |
|Cache    |20   |4    |1    |1,5        |4      |     |120      |24       |6        |
|k8s-cp   |50   |2    |2    |1,5        |3      |     |225      |9        |9        |
|k8s-node |100  |1    |1    |1,5        |3      |     |450      |4,5      |4,5      |
|         |     |     |     |           |       |     |         |         |         |
|         |     |     |     |           |       |     |952,5    |64,875   |85,5     |

### ИТОГО

Требуемые ресурсы:
- HDD:  ~ 1 TB
- RAM:  ~ 64 GB
- CPU:  ~ 96 Ядер

Кластер k8s:
- k8s Control Plane : 3 ноды
- k8s Nodes         : 3 ноды

P.S. Так как по условию задания часть параметров(HDD и CPU) не были обозначены, то были выбраны приблизительно-необходимые. 

---
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
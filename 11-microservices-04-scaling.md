# Домашнее задание к занятию "11.04 Микросервисы: масштабирование"

Вы работаете в крупной компанию, которая строит систему на основе микросервисной архитектуры. Вам как DevOps специалисту необходимо выдвинуть предложение по организации инфраструктуры, для разработки и эксплуатации.

---
---
### Задача 1: Кластеризация
Предложите решение для обеспечения развертывания, запуска и управления приложениями. Решение может состоять из одного или нескольких программных продуктов и должно описывать способы и принципы их взаимодействия.

Решение должно соответствовать следующим требованиям:

Поддержка контейнеров;
Обеспечивать обнаружение сервисов и маршрутизацию запросов;
Обеспечивать возможность горизонтального масштабирования;
Обеспечивать возможность автоматического масштабирования;
Обеспечивать явное разделение ресурсов доступных извне и внутри системы;
Обеспечивать возможность конфигурировать приложения с помощью переменных среды, в том числе с возможностью безопасного хранения чувствительных данных таких как пароли, ключи доступа, ключи шифрования и т.п.

Обоснуйте свой выбор.

---
### Ответ

Рассмотрим и сравним следующие решения, которые для удобства офрмим в таблицу ниже. 

| Критерий\Инструмент | Docker Swarm	| Nomand | Kubernetes | Apache Mesos (+Marathon, Aurora) |
|---|---|---|---|---|
| Поддержка контейнеров	| + | + | + | + |
| Обнаружение сервисов и маршрутизацию запросов | + | - | + | + |
| Возможность горизонтального масштабирования	| + | + | + | + |
| Возможность автоматического масштабирования	| - | + | + | + |
| Явное разделения ресурсов доступных извне и внутри системы	| - | + | + | + |
| Возможность конфигурирования приложения с помощью переменных среды, в том числе безопасное хранение чувствительных данных (пароли, ключи) | + | + | + | + |

- Решения все схожи.
- Выбор конктретного решения будет зависеть от ряда факторов: финансы, штат работников, их компетенции, документация, удобство использования, безопасность использования, поддержка сообществом и пр.
- Мой выбор - k8s. Наверное, сейчас самое распространенное решение.

---
---
### Задача 2: Распределенный кэш * (необязательная)

Разработчикам вашей компании понадобился распределенный кэш для организации хранения временной информации по сессиям пользователей. Вам необходимо построить Redis Cluster состоящий из трех шард с тремя репликами.

### Схема:
![11-04-01](https://user-images.githubusercontent.com/1122523/114282923-9b16f900-9a4f-11eb-80aa-61ed09725760.png)

---
### Ответ

---
---

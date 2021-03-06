# Домашнее задание к занятию "11.01 Введение в микросервисы" #

## Задача 1: Интернет Магазин ##

Руководство крупного интернет магазина у которого постоянно растёт пользовательская база и количество заказов рассматривает возможность переделки своей внутренней ИТ системы на основе микросервисов.

Вас пригласили в качестве консультанта для оценки целесообразности перехода на микросервисную архитектуру.

Опишите какие выгоды может получить компания от перехода на микросервисную архитектуру и какие проблемы необходимо будет решить в первую очередь.

### Выгоды (Достоинства) ###
* Удобная масштабируемость, которая требует меньше трудозатрат.
* Улучшенная безопасность программы, за счет того, что каждый отдельный модуль — это самостоятельный элемент программы. В случае выхода из строя или атаки на какой-нибудь модуль, вся остальная часть программы будет работать и находиться в безопасности.
* Снижение рисков деградации системы и полного отказа, за счет отсутствия единой точки отказа, повышение работоспособности.
* Отсутствие приверженности к определенному стеку технологий. Фактически, каждый отдельный модуль программы может быть разработан разными людьми и на разных языках программирования, что практически нереально при монолитном подходе.
* Более легкое обслуживание и понимание проекта. К примеру, если приходит в проект новый разработчик, то ему нужно изучить лишь модуль, за который он будет отвечать и над которым будет работать. Ему вовсе не нужно изучать весь код программы.
* Разделение на команды по компетенциям и зонам ответственности.
* Повторная реализация компонентов. То есть, разработав приложение на микросервисной архитектуре, в дальнейшем возможно использовать его отдельные модули в других своих разработках.

### Проблемы (Недостатки) ###
* Сложная распределенная система. Наличие множества отдельных сервисов влечет за собой дополнительную сложность в организационном и архитектурном плане. То есть усложняется контроль над различными командами разработчиков, а также само развертывание программного продукта.
* Изменения, затрагивающие несколько сервисов, должны координироваться между несколькими командами, а это может быть сложно, если команды еще не имели контактов.
* Усложненное тестирование. Потому что для начала нужно  будет тестировать каждый сервис отдельно, а потом совместную функциональность всех сервисов.
* Приложения на микросервисной архитектуре менее производительны, чем их аналоги на монолитной архитектуре за счет наличия множества баз данных и множества  более «длинных» путей связи между отдельными модулями.
* Более быстрая разработка и развертывание повлечет скорое устаревание документации.
* В связи с распределением систем и разделением комманд для разных сервисов, возможно, потребуется расширение штата разработчиков и инженеров сопровождения, а также повышение их компетенций и квалификации.
* Микросервисы — это дорого).

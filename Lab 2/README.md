# Лабораторная работа №2

Сегодня мы будем поднимать кластер, после чего в нем будет развернуто 2 сервиса:
 - Replication Сontroller ()
 - Deployment 
 
# Подготовка
После запуска Docker Engine из окна терминала был запущен Minikube командой:
> minikube start

После этого были написаны следующие файлы с расширением yaml, в которых указаны все данные необходимые для развертывания сервисов:

 - helloworld_rc.yaml
 - helloworld_dep.yaml

Код helloworld_rc.yaml

```
apiVersion: v1
kind: ReplicationController
metadata:
	labels:
		app: helloworld-v2
	name: helloworld-v2
spec:
	replicas: 1
	template:
		metadata:
			labels:
				app: helloworld-v2
			spec:
				containers:
				  - image: kelseyhightower/helloworld:v2
					name: helloworld-v2
					ports:
					  - containerPort: 8080
						name: http

---
apiVersion: v1
kind: Service
metadata:
	labels:
		app: helloworld-v2
	name: helloworld-v2
	namespace: default
spec:
	externalTrafficPolicy: Cluster
	internalTrafficPolicy: Cluster
	ipFamilyPolicy: SingleStack
	ports:
	- nodePort: 31438
	  port: 8080
	  protocol: TCP
	  targetPort: 8080
	selector:
		app: helloworld-v2
	sessionAffinity: None
	type: NodePort
status:
	loadBalancer: {}
```

Код helloworld_dep.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
	name: helloworld
spec:
	selector:
		matchLabels:
			app: helloworld
	replicas: 1
	template:
		metadata:
			labels:
				app: helloworld
		spec:
			containers:
			- name: helloworld
			  image: karthequian/helloworld:latest
			  ports:
			  - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
	name: helloworld
spec:
	selector:
		app: helloworld
	type: NodePort
	ports:
		- protocol: TCP
		  port: 80
		  targetPort: 80
```
 Для лабораторной работы был выбран именно NodePort как компонент контролирующий трафик, так как он весьма прост и используется на тестовых или легких проектах. Для этой лабораторной - идеальный выбор 👍

## Запуск

После приготовления оба манифеста встраиваются в кластер при помощи команд:
>kubectl apply -f helloworld_rc.yaml

>kubectl apply -f helloworld_dep.yaml

после чего идет запуск сервисов командами:
>minikube service hellowrold

>minikube servoce helloworld-v2

## Результаты работы

После запуска сервисов в терминале можно получить такой вывод для Replication Controller:
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-2-dev/Lab%202/base_lab/img/hw_rc_service.JPG"/></p>

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-2-dev/Lab%202/base_lab/img/hw_rc_page.JPG"/></p>

И следующий вывод для Deployment:

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-2-dev/Lab%202/base_lab/img/hw_dep_page.JPG"/></p>

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-2-dev/Lab%202/base_lab/img/hw_dep.JPG"/></p>

Мы большие молодцы, у нас всё работает😎

## Лабораторная 2✨

Для начала сгенерируем ключ и сертификат, как на рисунке ниже.

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-2-dev/Lab%202/star_task/star_img/gen_crt.JPG"/></p>

После этого генерируем секрет для kuber и прописываем его в манифесте для ingress и там же прописываем доменное имя хоста.

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-2-dev/Lab%202/star_task/star_img/gen_secret.JPG"/></p>

До пункта **2** я не мог додуматься и найти инфу 2 недели, так как я гордый владелец самой православной ОС в мире - Windows 10. Либо я *little silly boy*. После принятия манифестов нужно сделать 2 вещи: 
1. Зайти в файл в файл hosts и прописать "wbs.lab2"
2. Прописать в терминале minikube tunnel, чтобы создать сетевой маршрут к нашему сервису
После того как был сделан DOMAIN EXPANSION 🧙‍♂️- переходим по доменке сервиса в браузере и наконец таки видим рабочий сертификат.

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-2-dev/Lab%202/star_task/star_img/crt_and_kuber.JPG"/></p>

## Почему браузер ругается на самоподписанный сертификат?
Самоподписанные сертификаты - сертификаты, которые не выдаются организацией регистратором. Это во первых. Во вторых самоподписанный SSL-сертификат может быть создан любым человеком, и нет никакой возможности проверить, что он принадлежит именно веб-серверу, к которому происходит подключение. Вот такие пироги.

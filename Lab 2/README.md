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
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/tree/lab-2-dev/Lab%202/img/hw_rc_service.JPG"/></p>

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/tree/lab-2-dev/Lab%202/img/hw_rc_page.JPG"/></p>

И следующий вывод для Deployment:

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/tree/lab-2-dev/Lab%202/img/hw_dep.JPG"/></p>

<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/tree/lab-2-dev/Lab%202/img/hw_dep_page.JPG"/></p>

Мы большие молодцы, у нас всё работает😎

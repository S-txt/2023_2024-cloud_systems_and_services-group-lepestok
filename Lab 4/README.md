# Лабораторная работа №4

## Базовая
### *Prometheus*
Устанавливаем minikube на MacOS и запускаем его
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/1.JPG"/></p>
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/2.JPG"/></p>
Устанавливаем helm
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/3.JPG"/></p>
Далее добавим Prometheus Chart в нашу систему и добавим его в кластер кубера
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/45.JPG"/></p>
Посмотрим запущенные процессы в кластере и запущенные сервисы от прометеуса, они пригодятся дальше
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/67.JPG"/></p>
Предоставим доступ к службе прометеуса и запустим сервис
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/89.JPG"/></p>
Итог запуска промета
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/10.JPG"/></p>
### *Grafana*
добавляем графану в систему
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/11_12.JPG"/></p>
устанавливаем её в кластер и сразу проверяем запущенные службы
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/13_14.JPG"/></p>
запускаем сервис и сразу вычленяем сгенерированный пароль для учетки админа в графане
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/15_16.JPG"/></p>
Подключаемся к нашему кластеру 
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/18.JPG"/></p>
Импортируем и смотрим на дашборды с нашим кластером
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/20.JPG"/></p>
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/21.JPG"/></p>
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/22.JPG"/></p>


## Продвинутая
Создадим namespace для лаборторной работы:
```
kubectl create namespace lab4star
```
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/401.jpg"/></p>

### Запуск Prometheus
Развернем Prometheus с помощью файлов конфигураций:
```
kubectl create -f clusterRole.yaml
kubectl create -f config-map.yaml
kubectl create -f prometheus-deployment.yaml
kubectl create -f prometheus-service.yaml
kubectl create -f prometheus-ingress.yaml
```
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/406.jpg"/></p>

### Запуск Alert Manager
В качестве основы для конфигурации Alert Manager был взят tamplate из репозитория 
- AlertManagerConfigmap.yaml
    Указвается информация о канале связи. 
    
    В конфигурационный файл был записан token telegram бота созданный с помощью бота [@BotFather](https://t.me/BotFather)

    Так же в данном файле указан id канала, в который будет производиться рассылка алертов. Данный id можно получить с помощью бота [@RawDataBot](https://t.me/RawDataBot).

Последующие файлы были оставлены без изменений.



Далее запустим alertmanager:
```
kubectl create -f AlertManagerConfigmap.yaml
kubectl create -f AlertTemplateConfigMap.yaml
kubectl create -f Deployment.yaml
kubectl create -f Service.yaml
```
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/407.jpg"/></p>

Далее мы можем попасть в веб интерфейс Alert Manager пробросив порты с помощью команды:

```
kubectl port-forward -n <namespace>  <alert manager pod name> 9090:9093
```
Для задания алерта кодом, требуется в конфигурационный файл prometheus прописать правила уведомления:
```
  prometheus.rules: |-
    groups:
    - name: Test alert pod
      rules:
      - alert: Test pod load
        expr: sum(container_cpu_load_average_10s) > 1
        for: 1m
        labels:
          severity: telegram
        annotations:
          summary: Test pod load
```
В данном случае был взят аллерт срабатывающий при нагрузке на процессор в течении 10s более 1%.

В результате конфигурации peometheus через какое-то время в чат, указанный в конфигурации alert manager будет прислано уведомление о сработавшем алерте:
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-4-dev/Lab%204/img/409.jpg"/></p>

Вывод: В результате выполнения работы был изечен alert Manager, был развернут prometheus и настроен аллерт telegram с помощью кода

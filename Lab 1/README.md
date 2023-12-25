# Лабораторная работа 1
## Обычная
Для запуска контейнеров использовались команды `docker build -t lab_1_bad_practices`, `docker run -d lab_1_bad_practices` ("плохая" версия) и `docker build -t lab_1_good_practices`, `docker run -d -e port=80 lab_1_good_practices` ("хорошая" версия).
### "Плохие" практики
В качестве плохих практик в Dockerfile были использованы следующие четыре практики:

— `FROM ubuntu:latest` Использование последней версии образа Ubuntu без указания конкретной версии. Это может привести к несовместимости с зависимостями и непредсказуемому поведению в будущем.

— `RUN apt-get update && apt-get install -y apache2` Обновление пакетов и установка apache2 в одном слое. Это приводит к увеличению размера образа и усложняет отслеживание изменений.

— `CMD ["apache2ctl", "-D", "FOREGROUND"]` Использование CMD для запуска apache2 в фоновом режиме. Это может привести к тому, что контейнер будет считаться активным, даже если Apache не работает.

— `ARG port=80` Объявление переменных сред в описании Dockerfile.
### "Хорошие" практики
В "хорошей" версии Dockerfile все плохие практики были исправлены:

— `FROM ubuntu:20.04`  Указали конкретную версию образа Ubuntu для обеспечения стабильности и предсказуемости.

— `RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends apache2 \
    && rm -rf /var/lib/apt/lists/*` Разделили установку пакетов и обновления на два слоя для уменьшения размера образа и облегчения отслеживания изменений.

— `ENTRYPOINT ["apache2ctl", "-D", "FOREGROUND"]` Заменили ENTRYPOINT на CMD для запуска apache2, чтобы контейнер мог быть остановлен, если Apache не работает.

— Вместо явного указания переменной в Dockerfile вынесли установку переменной на уровень аргументов запуска контейнера.
### "Плохие" практики использования контейнера
В качестве примеров "плохого" использования контейнера можно назвать:

— Запуск контейнера с привилегиями root без необходимости. Это может привести к уязвимостям безопасности и компрометации хостовой системы.

— Использование устаревших версий пакетов или ядра. Это может привести к уязвимостям безопасности и несовместимости с другими приложениями.
## Со звездочкой
Для выполнения задачи был сформирован Docker Compose файл в котором были настроены MySQL как база данных и WordPress как источник данных. В параметрах WordPress были указаны данные о бд mysql.
```
version: "0.2"
services:
 db:
  image: mysql:5.7
  volumes:
   - db_data:/var/lib/mysql
  restart: always
  environment:
   MYSQL_ROOT_PASSWORD: rootpassword
   MYSQL_DATABASE: lab1
   MYSQL_USER: lab1_user
   MYSQL_PASSWORD: password
 wordpress:
  depends_on:
   - db
  image: wordpress:latest
  ports:
   - "8000:80"
  restart: always
  environment:
   WORDPRESS_DB_HOST: db:3306
   WORDPRESS_DB_USER: lab1_user
   WORDPRESS_DB_PASSWORD: password
   WORDPRESS_DB_NAME: lab1
volumes:
 db_data: {}
```

После запуска контейнеров мы можем зайти в веб интерфейс wordpress и зарегистрировать пользователя, данные о котором будут записаны в бд.
<p align="center"><img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-1-dev/Lab%201/img/1.jpg"/></p>

После перезагрузки контейнеров запись в бд будет сохранена, что и требовалось по условию работы.


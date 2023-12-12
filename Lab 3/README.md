# Лабораторная работа 3
## Обычная
Создадим репозиторий в GitLab (https://gitlab.com/aleksandrpiliakin/clouds):

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/repo/Lab%203/images/repo.png?raw=true" width=50% height=100%>

Репозиторий содержит простой веб-сервер на Express.js, который собирается в Docker, где Dockerfile имеет следующий вид:
```
FROM node:20-alpine
WORKDIR /app
COPY . /app/
RUN npm i
CMD ["npm", "run", "start"]
```

Для автоматической сборки и деплоя на сервер подключим CI/CD:

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/repo/Lab%203/images/ci-cd-runner.png?raw=true" width=50% height=100%>

Конфигурационный файл CI/CD для GitLab Runner следующий:
```
image: docker:latest

services:
  - docker:dind


stages:
  - build
  - deploy

build:
  stage: build
  script:
    - echo "build phase..."
    - docker build -t clouds-app .
    - docker login -u alekspi27 -p Sancho135720
    - docker tag clouds-app alekspi27/clouds:clouds-app
    - docker push alekspi27/clouds:clouds-app

deploy:
  stage: deploy
  before_script:
  #   - eval $(ssh-agent -s)
  #   - eval $(ssh-agent -s)
  #   - echo "$CURRENT_SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  #   - mkdir -p ~/.ssh
  #   - chmod 700 ~/.ssh
  #   - chmod 400 Clouds-lab.pem
    # - ssh-keyscan -p 22 -t rsa $SSH_HOST > ~/.ssh/known_hosts
    - chmod 400 Clouds-lab.pem
  script:
    - echo "deploy phase..."
    - docker login -u alekspi27 -p Sancho135720
    - docker pull alekspi27/clouds:clouds-app
    - ls
    - docker save alekspi27/clouds:clouds-app | gzip > clouds-app.tar.gz
    - scp -o StrictHostKeyChecking=no -i Clouds-lab.pem clouds-app.tar.gz ubuntu@ec2-54-210-230-237.compute-1.amazonaws.com:~/project/
    # - ssh -i -p 22 aleksandrpiliakin@95.25.207.154 chmod +x ./deploy
```

В нашем пайплане всего 2 стадии - `build` и `stages`, каждая из которых имеет по одноимённой джобе:

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/repo/Lab%203/images/pipeline.png?raw=true" width=50% height=100%>

Будем заливать файлы проекта на сервер и смотреть за исполнением паплайнов:

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/repo/Lab%203/images/pipelines.png?raw=true" width=50% height=100%>

Собранный Docker-образ после сборки сохраняется в приватный репозиторий Docker Registry, из которого читается на этапе деплоя. Во время деплоя архив с образом загружается на сервер, где его можно распаковать и запустить:

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/repo/Lab%203/images/deploy-result.png?raw=true" width=50% height=100%>


## Со звездочкой
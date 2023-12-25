# Лабораторная работа 3
## Обычная
Создадим репозиторий в GitLab (https://gitlab.com/aleksandrpiliakin/clouds):

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/Lab%203/images/repo.png?raw=true" width=50% height=100%>

Репозиторий содержит простой веб-сервер на Express.js, который собирается в Docker, где Dockerfile имеет следующий вид:
```
FROM node:20-alpine
WORKDIR /app
COPY . /app/
RUN npm i
CMD ["npm", "run", "start"]
```

Для автоматической сборки и деплоя на сервер подключим CI/CD:

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/Lab%203/images/ci-cd-runner.png?raw=true" width=50% height=100%>

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

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/Lab%203/images/pipeline.png?raw=true" width=50% height=100%>

Будем заливать файлы проекта на сервер и смотреть за исполнением паплайнов:

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/Lab%203/images/pipelines.png?raw=true" width=50% height=100%>

Собранный Docker-образ после сборки сохраняется в приватный репозиторий Docker Registry, из которого читается на этапе деплоя. Во время деплоя архив с образом загружается на сервер, где его можно распаковать и запустить:

<img src="https://github.com/S-txt/2023_2024-cloud_systems_and_services-group-lepestok/blob/lab-3-dev/Lab%203/images/deploy-result.png?raw=true" width=50% height=100%>


## Со звездочкой
В качестве хранилища для секретов использовался HashiCorp Vault. Лабораторная работа выполнялась на базе уже созданного и настроенного [репозитория в GitLab](https://gitlab.com/aleksandrpiliakin/clouds).
### Настройка HashiCorp Vault
Для начала необходимо с официального сайта [HashiCorp](https://developer.hashicorp.com/vault/install) установить приложение для поднятия сервера с хранилищем Vault для своей ОС (сайт доступен только с подключением прокси или VPN).

После установки поднимем сервер с хранилищем с помощью команды `vault server -dev`.

картинка

Обозначим переменные окружения адреса и токена хранилища следующими командами:
```
export VAULT_ADDR='https://127.0.0.1:8200'
export VAULT_TOKEN='hvs.5CZMotNlxaioL2sH3kQgCwi3'
```
Также для доступа GitLab к хранилищу необходимо прокинуть Vault-сервер с помощью утилиты Ngrok:

картинка

После нужно настроить метод аутентификации через JWT для нашего Vault-сервера. Сначала разрешим его командой `vault auth enable jwt`:

картинка

Теперь зададим метод аутентификации через JWT командой `vault write auth/jwt/config jwks_url="https://gitlab.com/-/jwks" bound_issuer="gitlab.com"`, где bound_issuer определяет, что только токены, выпущенные от имени домена gitlab.com могут использовать этот метод аутентификации, а страница https://gitlab.com/-/jwks должна использоваться в качестве точки валидации токена JWKS:

картинка

Создадим наши секреты. В качестве них у нас будут выступать логин и пароль от докера. Сделаем это следующими командами (реальные данные скрыты в целях конфиденциальности):
```
vault kv put secret/clouds/docker login=*login*
vault kv put secret/clouds/docker password=*password*
```
Чтобы проверить, что секреты были созданы успешно, можно посмотреть их значение командами:
```
vault kv get -field=login secret/clouds/docker*
vault kv get -field=password secret/clouds/docker*
```
Демонстрация работы команд для секрета с логином:

картинка
картинка

Пропишем политику безопасности, которая даст нам доступ на чтение секретов для докера:
```
vault policy write policy-clouds-docker - << EOF
path "secret/clouds/docker/*" {
  capabilities = ["read"]
}
EOF
```
Результат работы:

картинка

Теперь нам нужна роль, которая будет связывать созданную политику с JWT-токеном. Прописывается она следующим образом:
```
vault write auth/jwt/role/policy-clouds-docker - <<EOF
{
"role_type": "jwt",
"policies": ["policy-clouds-docker"],
"token_explicit_max_ttl": 60,
"user _claim": "deploy",
"bound _claims": {
"project_id": "26",
"ref": "master",
"ref_type": "branch"
}
}
EOF
```
Результат выполнения:

картинка

На этом настройка со стороны HashiCorp Vault завершена. 
### Настройка GitLab
Сначала убедимся, что включена функция, ограничивающая доступ к проекту из других джоб токенов:

картинка

Модифицируем наш .gitlab-ci.yml файл. Добавим стадию pre-build, где "достанем" наши секреты из хранилища и сохраним во временный файл build.env для использования в других джобах:
```
image: docker:latest

services:
  - docker:dind

stages:
  - pre-build
  - build
  - deploy

vault_prep:
  stage: pre-build
  image: hashicorp/vault:latest
  rules:
    - if: $CI_COMMIT_REF_NAME
      variables:
        VAULT_AUTH_ROLE: 'policy-clouds-docker'
  script:
    - echo "pre-build phase..."
    - export VAULT_ADDR=https://4074-95-25-211-92.ngrok-free.app
    - export VAULT_TOKEN=hvs.5CZMotNlxaioL2sH3kQgCwi3
    - echo $VAULT_AUTH_ROLE

    # Authenticate to the vault server
    # - export VAULT_TOKEN="$(vault write -field=token auth/jwt/login role=$VAULT_AUTH_ROLE jwt=$CI_JOB_JWT)"

    # Attempt to get the staging password - only works on `main`
    - export DOCKER_PASSWORD="$(vault kv get -field=password secret/clouds/docker)"
    - export DOCKER_LOGIN="$(vault kv get -field=login secret/clouds/docker)"
    - echo $DOCKER_LOGIN > build.env
    - echo $DOCKER_PASSWORD | tee -a  build.env > /dev/null
  artifacts:
    paths:
      - build.env
      

build:
  stage: build
  needs: ["vault_prep"]
  script:
    - echo "build phase..."
    - cat build.env
    - DOCKER_LOGIN=$(head -n 1 build.env)
    - DOCKER_PASSWORD=$(tail -n 1 build.env)
    - docker build -t clouds-app .
    - echo $DOCKER_LOGIN
    - echo $DOCKER_PASSWORD
    - docker login -u $DOCKER_LOGIN -p $DOCKER_PASSWORD
    - docker tag clouds-app $DOCKER_LOGIN/clouds:clouds-app
    - docker push $DOCKER_LOGIN/clouds:clouds-app

deploy:
  stage: deploy
  needs: ["vault_prep"]
  before_script:
  #   - eval $(ssh-agent -s)
  #   - eval $(ssh-agent -s)
  #   - echo "$CURRENT_SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
  #   - mkdir -p ~/.ssh
  #   - chmod 700 ~/.ssh
  #   - chmod 400 Clouds-lab.pem
    # - ssh-keyscan -p 22 -t rsa $SSH_HOST > ~/.ssh/known_hosts
    - chmod 400 Clouds-lab.pem
  rules:
    - if: $CI_COMMIT_REF_NAME
      variables:
        VAULT_AUTH_ROLE: 'policy-clouds-docker'
  script:
    - cat build.env 
    - DOCKER_LOGIN=$(head -n 1 build.env)
    - DOCKER_PASSWORD=$(tail -n 1 build.env)
    - docker login -u $DOCKER_LOGIN -p $DOCKER_PASSWORD
    - docker pull $DOCKER_LOGIN/clouds:clouds-app
    - ls
    - docker save $DOCKER_LOGIN/clouds:clouds-app | gzip > clouds-app.tar.gz
    - scp -o StrictHostKeyChecking=no -i Clouds-lab.pem clouds-app.tar.gz ubuntu@ec2-54-210-230-237.compute-1.amazonaws.com:~/project/
    # - ssh -i -p 22 aleksandrpiliakin@95.25.207.154 chmod +x ./deploy
```
> Авторы понимают, что сохранение секретов в файл не является безопасным, однако в текущей реализации процесса CI/CD проекта это единственный вариант, который работает, как задумано. Настройка более продвинутой версии CI/CD не является целью лабораторной, целью является лишь демонстрация возможности самой работы CI/CD в GitLab, а также интеграции с Vault

Все стадии пайплайна отрабатывают успешно:

картинка

В итоге, как и в Лабораторной работе 3, мы можем убедиться, что образ проекта загружен на тестовый сервер:

картинка


# Ежедневник для записи заданий на день
### Что умеет проект?
Пользователь редактируется и имеет личный кабинет, где он может:
Добавлять, просматривать, редактировать и удалять задания.
Добавлять задания в категории "сделано" или "не сделано".

#### Основной стек:
- Python 3
- Django
- Django REST Framework
- Nginx
- Docker
- CI/CD (GitHub Actions)

#### Установка
- Клонируйте репозиторий на свой компьютер:
```bash
git clone git@github.com:Vlaldis95/taski_docker.git
```
```bash
cd kittygram
```
- Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.example.

#### Создание Docker-образов
- Замените username на ваш логин на DockerHub:
```bash
cd frontend
docker build -t username/taski_frontend .
cd ../backend
docker build -t username/taski_backend .
cd ../nginx
docker build -t username/taski_gateway .
```
#### Загрузите образы на DockerHub:
```bash
docker push username/taski_frontend
docker push username/taski_backend
docker push username/taski_gateway
```
#### Деплой на сервере

- Подключитесь к удаленному серверу
```bash
ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера
```
- Создайте на сервере директорию taski через терминал
```bash
mkdir taski
```
#### Установка docker compose на сервер:
```bash
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin

- В директорию taski/ скопируйте файлы docker-compose.production.yml и .env:
```bash
scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/taski/docker-compose.production.yml
* ath_to_SSH — путь к файлу с SSH-ключом;
* SSH_name — имя файла с SSH-ключом (без расширения);
* username — ваше имя пользователя на сервере;
* server_ip — IP вашего сервера.
```
- Запустите docker compose в режиме демона:
```bash
sudo docker compose -f docker-compose.production.yml up -d
```
- Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
``` bash
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```
- На сервере в редакторе nano откройте конфиг Nginx:
```bash
sudo nano /etc/nginx/sites-enabled/default
```
- Измените настройки location в секции server:
```bash
location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:8000;
}
```
- Проверьте работоспособность конфига Nginx:
```bash
sudo nginx -t
```
- Если ответ в терминале такой, значит, ошибок нет:
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
- Перезапускаем Nginx
```bash
sudo service nginx reload
```
### Настройка CI/CD
Файл workflow уже написан. Он находится в директории taski/.github/workflows/main.yml
Для адаптации его на своем сервере добавьте секреты в GitHub Actions:
```bash
DOCKER_USERNAME                # имя пользователя в DockerHub
DOCKER_PASSWORD                # пароль пользователя в DockerHub
HOST                           # ip_address сервера
USER                           # имя пользователя
SSH_KEY                        # приватный ssh-ключ (cat ~/.ssh/id_rsa)
SSH_PASSPHRASE                 # кодовая фраза (пароль) для ssh-ключа
TELEGRAM_TO                    # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
TELEGRAM_TOKEN                 # токен бота (получить токен можно у @BotFather, /token, имя бота)
```

### Как проверить работу с помощью автотестов
- В корне репозитория создайте файл tests.yml со следующим содержимым:
```
repo_owner: ваш_логин_на_гитхабе
taski_domain: полная ссылка (https://доменное_имя) на ваш проект Taski
dockerhub_username: ваш_логин_на_докерхабе
```
- Скопируйте содержимое файла .github/workflows/main.yml в файл kittygram_workflow.yml в корневой директории проекта.
- Для локального запуска тестов создайте виртуальное окружение, установите в него зависимости из backend/requirements.txt и запустите в корневой директории проекта pytest.
- 
Чек-лист для проверки перед отправкой задания
Проект Taski доступен по доменному имени, указанному в tests.yml.
Пуш в ветку main запускает тестирование и деплой Taski, а после успешного деплоя вам приходит сообщение в телеграм.
В корне проекта есть файл taski_workflow.yml.

## Автор
:white_check_mark: Владислав Лукьяненко (https://github.com/Vlaldis95)

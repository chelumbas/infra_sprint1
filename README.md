# Kittygram

## Описание
Социальная сеть для обмена фотографиями любимых питомцев.

## Установка

В примерах описана установка на Ubuntu Linux.

## Подготовка к локальному запуску

- Копировать репозиторий на локальную машину.
```git clone git@github.com:chelumbas/infra_sprint1.git```
- Создать виртуальное окружние.
```python3 -m venv venv```
- Активировать виртуальное окружение.
```source venv/bin/activate```
- Установить зависимости для проекта.
```pip install -r requirements.txt```
- Выполнить миграции.
```python infra_sprint1/backend/manage.py migrate```
- Создать суперпользователя с кредами для его использования.
```python infra_sprint1/backend/manage.py createsuperuser```
- В настройках Django-проекта по адресу ```nfra_sprint1/backend/kittygram_backend/settings.py``` прописать адреса на которых будет работать сервис, расположения медиа-файлов.
- Скачать и установить Node.js.
```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - && sudo apt-get install -y nodejs```
- Перейти по адресу ```infra_sprint1/frontend``` и установить зависимости.
```npm i```
- Для запуска backend на определенном порту установить gunicorn.
```pip install gunicorn```

Проверить, что все корректно работает можно командой:
```gunicorn --bind 0.0.0.0:8000 kittygram_backend.wsgi```
Сервис должен запуститься и быть доступен на 8000 порту.

## Настройка автозапуска
- Создать файл из под root ```gunicorn.service``` по адресу ```/etc/systemd/system/```

Пример конфигурации файла:
```
[Unit]
Description=gunicorn daemon 
After=network.target

[Service]
User=yc-user 
WorkingDirectory=АДРЕС ДИРЕКТОРИИ С ФАЙЛОМ_manage.py
ExecStart=/home/ИМЯ ПОЛЬЗОВАТЕЛЯ/АДРЕС gunicorn В ВИРТУАЛЬНОМ ОКРУЖЕНИИ --bind 0.0.0.0:8080 kittygram_backend.wsgi

[Install]
WantedBy=multi-user.target
```
- Запустить службу.
```systemctl start gunicorn```
- Добавить службу в автозапуск.
```systemctl enable gunicorn```

## Работа через веб-сервер

- Установить NGINX.
```apt install nginx```
- Запустить Nginx.
```systemctl start nginx```
- Настроить дефолтный фаервол.
```ufw allow 'Nginx Full'``` - доступ по портам Nginx
```ufw allow OpenSSH``` - доступ по SSH
- Включить фаервол.
```ufw enable```
- Прописать настройки сервиса для Nginx в файле ```/etc/nginx/sites-enabled/default```

Пример конфигурации:
```
server {

    server_name server_name <IP адрес сервера> <доменное имя>;

    location /api/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /admin/ {
        proxy_pass http://127.0.0.1:8000;
    }

    location /media/ {
        alias /var/www/infra_sprint1/media/;
    }

    location / {
    root    /var/www/infra_sprint1;
    index   index.html index.htm;
    try_files  $uri /index.html;
    }

}
```
- Проверить, что конфигурация составлена верно.
```nginx -t```
- Перезапустить Nginx.
```systemctl reload nginx```

## Настройка HTTPS-соединения
- Установить пакетный менеджер snapd.
```apt install snapd```
- Установить зависимости.
```snap install core; snap refresh core```
- Установить sertbot.
```snap install --classic certbot```
- С помощью sertbot выпустить сертификаты для Nginx
```certbot --nginx```

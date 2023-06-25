# Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.
## Описание проекта.
Проект написан в рамках учебного курса по Python от Яндекс.Практикум. Пользователи могут регистрироваться, загружать фотографии своих котов с кратким описанием, и смотреть котиков других пользователей. Основная задача - обучение деплою проекта на сервер.

## Технологии.
* Python 3.9
* Django==3.2.3
* djangorestframework==3.12.4
* Nginx
* Gunicorn

#### Установка проекта на локальный компьютер из репозитория.
Клонировать репозиторий git clone <адрес вашего репозитория>.
Перейти в директорию с клонированным репозиторием.
Установить виртуальное окружение ```python3 -m venv venv```.
Установить зависимости ```pip install -r requirements.txt```.
В директории ```/backend/kittygram_backend/``` создать файл ```.env```.
В файле ```.env``` прописать ваш ```SECRET_KEY``` в виде: 
```SECRET_KEY = '<ваш_ключ>' и ALLOWED_HOSTS = '['localhost']'```

#### Деплой проекта на удаленный сервер.
Подключение сервера к аккаунту на GitHub, на сервере должен быть установлен Git.
Для проверки выполнить ```sudo apt update git --version```.
Если Git не установлен - установить командой ```sudo apt install git```.
Находясь на сервере сгенерировать пару SSH-ключей командой ssh-keygen.
Сохранить открытый ключ в вашем аккаунте на GitHub. Для этого вывести ключ в терминал командой ```cat .ssh/id_rsa.pub```. Скопировать ключ от символов ssh-rsa, включительно, и до конца. Добавить это ключ к вашему аккаунту на GitHub.
Клонировать проект с GitHub на сервер: ```git clone git@github.com:Ваш_аккаунт/<Имя проекта>.git```.

#### Запуск backend проекта на сервере.
Установить пакетный менеджер и утилиту для создания виртуального окружения ```sudo apt install python3-pip python3-venv -y```.
Находясь в директории с проектом создать и активировать виртуальное окружение ```python3 -m venv venv source venv/bin/activate```.
Установить зависимости ```pip install -r requirements.txt```.
Выполнить миграции ```python manage.py migrate```.
Создать суперюзера ```python manage.py createsuperuser```.
Отредактировать .env на сервере: в список ```ALLOWED_HOSTS``` добавить внешний IP-адрес вашего сервера и адреса ```127.0.0.1``` и ```localhost``` . ```ALLOWED_HOSTS = ['<внешний адрес вашего сервера>', '127.0.0.1', 'localhost']```

#### Запуск frontend проекта на сервере.
Установить на сервер ```Node.js``` командами ```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\ sudo apt-get install -y nodejs```
Установить зависимости frontend приложения. Из директории <ваш проект>/frontend/ выполнить команду: ```npm i```

#### Установка и запуск Gunicorn.
При активированном виртуальном окружении проекта установить пакет ```gunicorn pip install gunicorn==20.1.0```
Открыть файл settings.py проекта и установить для константы DEBUG значение False ```DEBUG = False```
В директории /etc/systemd/system/ создайте файл gunicorn.service ```sudo nano /etc/systemd/system/gunicorn.service``` со следующим кодом (без комментариев):

    [Unit]
    Description=gunicorn daemon
    After=network.target
    
    [Service]
    User=yc-user
    WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/
    ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    
    [Install]
    WantedBy=multi-user.target

Чтобы точно узнать путь до Gunicorn можно при активированном виртуальном окружении использовать команду ```which gunicorn```

#### Установка и настройка Nginx.
На сервере из любой директории выполнить команду: ```sudo apt install nginx -y```
Для установки ограничений на открытые порты выполнить по очереди команды: ```sudo ufw allow 'Nginx Full'``` 
```sudo ufw allow OpenSSH```
Включить файервол ```sudo ufw enable```

#### Cобрать и разместить статику frontend-приложения.
Перейти в директорию <имя_проекта>/frontend/ и выполнить команду ```npm run build```
результат сохранится в директории .../frontend/build/. В системную директорию сервера /var/www/ скопировать содержимое папки /frontend/build/
Открыть файл конфигурации веб-сервера sudo nano /etc/nginx/sites-enabled/default и заменить его содержимое следующим кодом:

    server { 
        listen 80;
        server_name публичный_ip_вашего_удаленного_сервера;
        
        location / {
            root   /var/www/<имя проекта>;
            index  index.html index.htm;
            try_files $uri /index.html;
        }
    }  

Проверить корректность конфигурации ```sudo nginx -t```
Перезагрузить конфигурацию Nginx ```sudo systemctl reload nginx```

#### Настроить проксирование запросов.
Открыть файл конфигурации Nginx /etc/nginx/sites-enabled/default и добавить в него ещё один блок location

    server {
        listen 80;
        server_name публичный_ip_вашего_удаленного_сервера;
        
        location /api/ {
            proxy_pass http://127.0.0.1:8080;
        }
        location /admin/ {
            proxy_pass http://127.0.0.1:8000;
        }
        location / {
            root   /var/www/<имя_проекта>;
            index  index.html index.htm;
            try_files $uri /index.html;
        }
    }

Сохранить изменения, проверить и перезагрузить конфигурацию веб-сервера:

  ```sudo nginx -t```
  ```sudo systemctl reload nginx```
Собрать и настроить статику для backend-приложения.
В файле settings.py прописать настройки

  ```STATIC_URL = 'static_backend'```
  ```STATIC_ROOT = BASE_DIR / 'static_backend'```
Активировать виртуальное окружение проекта, перейти в директорию с файлом manage.py и выполнить команду ```python manage.py collectstatic```

В директории_<имя_проекта>/backend/_ будет создана директория static_backend/
Скопировать директорию static_backend/ в директорию /var/www/<имя_проекта>/

#### Добавление доменного имени в настройки Django.
В файле ```.env``` добавить в список ALLOWED_HOSTS доменное имя: 
```ALLOWED_HOSTS = ['ip_адрес_вашего_сервера', '127.0.0.1', 'localhost', 'ваш-домен']```

Сохранить изменения и перезапустить gunicorn ```sudo systemctl restart gunicorn```
Внести изменения в конфигурацию Nginx. Открыть конфигурационный файл командой: 
```sudo nano /etc/nginx/sites-enabled/default```

Добавьте в строку server_name выданное вам доменное имя (через пробел, без < >):

    server {
        ...
          server_name <ваш-ip> <ваш-домен>;
        ...
    }
Проверить конфигурацию ```sudo nginx -t``` и перезагрузить её командой ```sudo systemctl reload nginx```, чтобы изменения вступили в силу.

#### Получение и настройка SSL-сертификата.
**Установка certbot**
Зайдите на сервер и последовательно выполните команды:
 ```sudo apt install snapd```
 ```sudo snap install core; sudo snap refresh core```
 ```sudo snap install --classic certbot```
 ```sudo ln -s /snap/bin/certbot /usr/bin/certbot```
Запустить certbot и получить SSL-сертификат:

  ```sudo certbot --nginx```
Сертификат автоматически сохранится на вашем сервере в системной директории /etc/ssl/ Также будет автоматически изменена конфигурация Nginx: в файл /etc/nginx/sites-enabled/default добавятся новые настройки и будут прописаны пути к сертификату.
Перезагрузить конфигурацию Nginx ```sudo systemctl reload nginx```


#### Автор: uHDezuT

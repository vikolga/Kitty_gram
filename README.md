# infra_sprint1

Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

Деплой учебного проекта.

Установка проекта на локальный компьютер из репозитория Клонировать репозиторий https://github.com/vikolga/infra_sprint1.git перейти в директорию с клонированным репозиторием

Cоздать и активировать виртуальное окружение:

python3 -m venv venv (для mac и linux) python -m venv venv (для windows) source venv/bin/activate (для mac и linux) source venv/Scripts/activate (для windows) Установить зависимости из файла requirements.txt:

python3 -m pip install --upgrade pip (для mac и linux) python -m pip install --upgrade pip (для windows) перейти в дирректорию где хранится файл requirements.txt и оттуда выполнить команду:

pip install -r requirements.txt Выполнить миграции:

python3 manage.py migrate (для mac и linux) python manage.py migrate (для windows)

в директории /backend/kittygram_backend/ создать файл .env в файле .env прописать ваш SECRET_KEY в виде: SECRET_KEY = '<ваш_ключ>'

Деплой проекта на удаленный сервер Подключение сервера к аккаунту на GitHub на сервере должен быть установлен Git. Для проверки выполнить sudo apt update git --version если Git не установлен - установить командой sudo apt install git находясь на сервере сгенерировать пару SSH-ключей командой ssh-keygen сохранить открытый ключ в вашем аккаунте на GitHub. Для этого вывести ключ в терминал командой cat .ssh/id_rsa.pub. Скопировать ключ от символов ssh-rsa, включительно, и до конца. Добавить это ключ к вашему аккаунту на GitHub. клонировать проект с GitHub на сервер: git clone git@github.com:Ваш_аккаунт/<Имя проекта>.git

Запуск backend проекта на сервере Установить пакетный менеджер и утилиту для создания виртуального окружения sudo apt install python3-pip python3-venv -y находясь в директории с проектом создать и активировать виртуальное окружение python3 -m venv venv source venv/bin/activate установить зависимости pip install -r requirements.txt выполнить миграции python manage.py migrate создать суперюзера python manage.py createsuperuser отредактировать settings.py на сервере: в список ALLOWED_HOSTS добавить внешний IP-адрес вашего сервера и адреса 127.0.0.1 и localhost . ALLOWED_HOSTS = ['<внешний адрес вашего сервера>', '127.0.0.1', 'localhost']

Запуск frontend проекта на сервере установить на сервер Node.js командами curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\ sudo apt-get install -y nodejs установить зависимости frontend приложения. Из директории <ваш проект>/frontend/ выполнить команду: npm i

Установка и запуск Gunicorn при активированном виртуальном окружении проекта установить пакет gunicorn pip install gunicorn==20.1.0

открыть файл settings.py проекта и установить для константы DEBUG значение False DEBUG = False

В директории /etc/systemd/system/ создайте файл gunicorn.service sudo nano /etc/systemd/system/gunicorn.service со следующим кодом (без комментариев):

[Unit]

Description=gunicorn daemon

After=network.target

[Service]

User=yc-user

WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/

ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi

[Install]

WantedBy=multi-user.target Чтобы точно узнать путь до Gunicorn можно при активированном виртуальном окружении использовать команду which gunicorn

Установка и настройка Nginx На сервере из любой директории выполнить команду: sudo apt install nginx -y

Для установки ограничений на открытые порты выполнить по очереди команды: sudo ufw allow 'Nginx Full' sudo ufw allow OpenSSH

включить файервол sudo ufw enable

собрать и разместить статику frontend-приложения. Перейти в директорию <имя_проекта>/frontend/ и выполнить команду npm run build Результат сохранится в директории .../frontend/build/. В системную директорию сервера /var/www/ скопировать содержимое папки /frontend/build/

открыть файл конфигурации веб-сервера sudo nano /etc/nginx/sites-enabled/default и заменить его содержимое следующим кодом:

server {

  listen 80;
  server_name публичный_ip_вашего_удаленного_сервера;

  location / {
  root   /var/www/<имя проекта>;
  index  index.html index.htm;
  try_files $uri /index.html;
  }
} проверить корректность конфигурации sudo nginx -t

перезагрузить конфигурацию Nginx sudo systemctl reload nginx

настроить проксирование запросов Открыть файл конфигурации Nginx /etc/nginx/sites-enabled/default и добавить в него ещё один блок location

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
} Сохранить изменения, проверить и перезагрузить конфигурацию веб-сервера:

sudo nginx -t sudo systemctl reload nginx собрать и настроить статику для backend-приложения. в файле settings.py прописать настройки

STATIC_URL = 'static_backend' STATIC_ROOT = BASE_DIR / 'static_backend' активировать виртуальное окружение проекта, перейти в директорию с файлом manage.py и выполнить команду python manage.py collectstatic

в директории_<имя_проекта>/backend/_ будет создана директория static_backend/

Скопировать директорию static_backend/ в директорию /var/www/<имя_проекта>/

Добавление доменного имени в настройки Django в файле settings.py добавить в список ALLOWED_HOSTS доменное имя: ALLOWED_HOSTS = ['ip_адрес_вашего_сервера', '127.0.0.1', 'localhost', 'ваш-домен']

сохранить изменения и перезапустить gunicorn sudo systemctl restart gunicorn

внести изменения в конфигурацию Nginx. Открыть конфигурационный файл командой: sudo nano /etc/nginx/sites-enabled/default

Добавьте в строку server_name выданное вам доменное имя (через пробел, без < >):

server { ... server_name <ваш-ip> <ваш-домен>; ... } Проверить конфигурацию sudo nginx -t и перезагрузить её командой sudo systemctl reload nginx, чтобы изменения вступили в силу.

Получение и настройка SSL-сертификата Установка certbot

Зайдите на сервер и последовательно выполните команды:

sudo apt install snapd sudo snap install core; sudo snap refresh core sudo snap install --classic certbot sudo ln -s /snap/bin/certbot /usr/bin/certbot запустить certbot и получить SSL-сертификат:

sudo certbot --nginx сертификат автоматически сохранится на вашем сервере в системной директории /etc/ssl/ Также будет автоматически изменена конфигурация Nginx: в файл /etc/nginx/sites-enabled/default добавятся новые настройки и будут прописаны пути к сертификату.

перезагрузить конфигурацию Nginx sudo systemctl reload nginx

Автор Викторова Ольга

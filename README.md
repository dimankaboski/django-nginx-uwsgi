# django-nginx-uwsgi

> OS: Ubuntu 16.04x86_64
###### files:
- 'mysite_nginx.conf': simple nginx config
- 'mysite_uwsgi.ini': simple uwsgi config
- 'uwsgi_params': uwsgi params


main post: 'https://habr.com/ru/post/226419/'

Проверка портов: 

    sudo lsof -nP -i | grep LISTEN

## 1. Remove apache2

    service apache2 stop
    sudo apt remove apache2.*
    sudo apt-get purge apache2 apache2-utils apache2.2-bin apache2-common
        //or
    sudo apt-get purge apache2 apache2-utils apache2-bin apache2.2-common
    sudo apt-get autoremove

## 2. Install pip3

    sudo apt-get update
    sudo apt-get install pythonX.Y-dev
    sudo apt-get -y install python3-pip
    pip3 install virtualenv

## 2. Install python3.7 (Если всё плохо)

    sudo apt-get update
    sudo apt install software-properties-common
    sudo add-apt-repository ppa:deadsnakes/ppa
    sudo apt update
    sudo apt install python3.7
    
## 2.0 порядок запуска для питона (python 3.7 будет запускаться заместо 3.5)

    sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.7 1
    sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.5 2
    sudo update-alternatives --config python3

### 2.1 Установка дополнительных инструментов

    sudo apt install -y build-essential libssl-dev libffi-dev python3-dev
    
### 2.2 Install python3.7 venv

    sudo apt install python3.7-venv
    python3 -m venv py37-venv
  
## 3. Install nginx, wsgi etc

    
    sudo apt-get install nginx
    sudo service nginx start
    sudo apt-get install git
    pip3 install uwsgi


## 4. Add sudo users

    sudo adduser newuser
    sudo usermod -a -G www-data username

## 5. Create virtualenv and clone project

    virtualenv new-env
    cd new-env
    git clone
    source bin/activate

В созданной виртуальной среде устанавливаем всё необходимое для работы приложения

    pip3 install uwsgi
    pip3 install ...

Проверяем работу приложения

    python3 manage.py runserver

## 6. Конфигурация nginx для работы с Django

- Нам понадобится файл 'uwsgi_params'. Ставим его в корневую папку нашего проекта.

- Ставим файл mysite_nginx.conf в корневую папку проекта (Не забудьте поменять пути).

В настройка nginx /etc/nginx/nginx.conf в разделе http прописываем

    include /etc/nginx/sites-enabled/*;

Меняем

    user nginx;

на

    user www-data;

Создаем dir sites-enabled в /etc/nginx/

    sudo mkdir /etc/nginx/sites-enabled

В папке /etc/nginx/sites-enabled создаем ссылку на файл mysite_nginx.conf, чтобы nginx увидел его:

    sudo ln -s /path/to/your/mysite/mysite_nginx.conf /etc/nginx/sites-enabled/

## 7. Проверка осблуживания статики и медиа

Перезапускаем nginx: 

    sudo service nginx restart

В браузере переходим по адресу yourserver.com:8000/media/media.png и, если видим наш файл, значит мы все сделали правильно.

## 8. Конфигурация uWSGI через ini файл

В корневую папку проекта ставим 'mysite_uwsgi.ini'

Запускаем uWSGI:

    sudo uwsgi --ini mysite_uwsgi.ini

## 9. Режим Emperor

Создаем папку для конфигурационных файлов:

    sudo mkdir /etc/uwsgi
    sudo mkdir /etc/uwsgi/vassals

Создаем в ней ссылку на mysite_uwsgi.ini:

    sudo ln -s /path/to/your/mysite/mysite_uwsgi.ini /etc/uwsgi/vassals/

Запускаем uWSGI в режиме Emperor:

    sudo uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

Опции:
- emperor: папка с конфигурационными файлами
- uid: id пользователя, от имени которого будет запущен процесс
- gid: id группы, от имени которой будет запущен процесс

Проверяем. yourserver.com/

## 10. Автоматический запуск uWSGI после загрузки операционной системы

В файл /etc/rc.local, перед строкой “exit 0” добавляем:

    /usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

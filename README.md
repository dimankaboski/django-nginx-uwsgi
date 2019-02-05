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

    sudo apt-get -y install python3-pip
    pip3 install virtualenv


## 3. Install nginx, wsgi etc

    sudo apt-get install pythonX.Y-dev
    sudo apt-get install nginx
    sudo service nginx start
    pip3 install uwsgi


## 4. Add sudo users

    sudo adduser newuser
    usermod -aG sudo newuser

## 5. Create virtualenv and clone project

    virtualenv new-env

## 6. Проверка

Создаем файл test.py:

    #test.py
    def application(env, start_response):
        start_response('200 OK', [('Content-Type','text/html')])
        return [b"Hello World"] # python3
        #return ["Hello World"] # python2

Запускаем uWSGI:

    uwsgi --http :8000 --wsgi-file test.py


В браузере переходим по адресу yourserver.com:8000.
Видим: «Hello, world», значит, мы все сделали правильно и следующие компоненты работают:

    Пользователь <-> uWSGI <-> test.py 

## 7. Проверка работы Django приложения

Проверим, что только что созданный проект mysite запускается на сервере для разработки:

    python manage.py runserver 0.0.0.0:8000

Если проект запустился, останавливаем сервер для разработки и запускаем uWSGI следующим образом:

    uwsgi --http :8000 --module mysite.wsgi

- module mysite.wsgi: uwsgi загрузит модуль mysite.wsgi

В браузере переходим по адресу yourserver.com:8000.
Видим стартовую страницу Djangо, значит, мы все сделали правильно и следующие компоненты работают:

    Пользователь <-> uWSGI <-> Django

## 8. Конфигурация nginx для работы с Django

- Нам понадобится файл 'uwsgi_params'. Ставим его в корневую папку нашего проекта.

- Ставим файл mysite_nginx.conf в корневую папку проекта (Не забудьте поменять пути).

В папке /etc/nginx/sites-enabled создаем ссылку на файл mysite_nginx.conf, чтобы nginx увидел его:

    sudo ln -s ~/path/to/your/mysite/mysite_nginx.conf /etc/nginx/sites-enabled/

## 9. Проверка осблуживания статики и медиа

Перезапускаем nginx: 

    sudo service nginx restart

В браузере переходим по адресу yourserver.com:8000/media/media.png и, если видим наш файл, значит мы все сделали правильно.

## 10. nginx + uWSGI + test.py

Запускаем uWSGI:

    uwsgi --socket mysite.sock --wsgi-file test.py

На этот раз опция socket указывает на файл.
Открываем в браузере yourserver.com:8000/

## Если не заработало

Проверьте лог ошибок nginx, скорее всего он находится в файле var/log/nginx/error.log
Если найдете там что-то похожее на

    connect() to unix:///path/to/your/mysite/mysite.sock failed (13: Permission denied)

значит есть проблема с правами доступа к файлу mysite.sock. Необходимо сделать так, чтобы nginx имел разрешение на использование этого файла.

Попробуйте запустить uWSGI так: 

    uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=666 #много полномочий

Или так:

    uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=664 #более разумно

Чтобы проблем с доступом в будущем не было, добавьте вашего пользователя в группу www-data.

Информация, которую uWSGI выводит в терминал, полезна при поиске и исправлении возможных ошибок или неисправностей.

## 11. nginx + uWSGI + Django

Запускаем:

    uwsgi --socket mysite.sock --module mysite.wsgi --chmod-socket=664

В браузере переходим на yourserver.com:8000/ и видим стартовую страницу Django.

    Пользователь <-> Веб-сервер <-> Сокет <-> uwsgi <-> Django

Мы собрали всю цепочку, но настройка еще не закончена, идем дальше.

## 12. Конфигурация uWSGI через ini файл

В корневую папку проекта ставим 'mysite_uwsgi.ini'

Запускаем uWSGI:

    uwsgi --ini mysite_uwsgi.ini

## 13. Режим Emperor

Создаем папку для конфигурационных файлов:

    sudo mkdir /etc/uwsgi
    sudo mkdir /etc/uwsgi/vassals

Создаем в ней ссылку на mysite_uwsgi.ini:

    sudo ln -s /path/to/your/mysite/mysite_uwsgi.ini /etc/uwsgi/vassals/

Запускаем uWSGI в режиме Emperor:

    sudo uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

Опции:
- emperor: папка с конфигурациолнными файлами
- uid: id пользователя, от имени которого будет запущен процесс
- gid: id группы, от имени которой будет запущен процесс

Проверяем. yourserver.com:8000/

## 14. Автоматичесеий запуск uWSGI после загрузки операционной системы

В файл /etc/rc.local, перед строкой “exit 0” добавляем:

    /usr/local/bin/uwsgi --emperor /etc/uwsgi/vassals --uid www-data --gid www-data

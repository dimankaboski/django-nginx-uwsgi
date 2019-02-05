# django-nginx-uwsgi

> OS: Ubuntu 16.04x86_64

files:

    1. 'mysite_nginx.conf': simple nginx config
    2. 'mysite_uwsgi.ini': simple uwsgi config
    3. 'uwsgi_params': uwsgi params



проверка портов: sudo lsof -nP -i | grep LISTEN


## 1. Remove apache2

'''

## 1. Remove apache2

'''

    service apache2 stop
    sudo apt remove apache2.*
    sudo apt-get purge apache2 apache2-utils apache2.2-bin apache2-common
        //or
    sudo apt-get purge apache2 apache2-utils apache2-bin apache2.2-common
    sudo apt-get autoremove
    
'''

'''

## 2. Install pip3

'''

    sudo apt-get -y install python3-pip
    pip3 install virtualenv
    
'''

###### 3. Install nginx, wsgi etc

'''

## 3. Install nginx, wsgi etc

'''

    sudo apt-get install pythonX.Y-dev
    sudo apt-get install nginx
    pip3 install uwsgi
    
'''

###### 4. Add sudo users

'''

## 4. Add sudo users

'''

    sudo adduser newuser
    usermod -aG sudo newuser
    
'''

'''

## 5. Create virtualenv and clone project

'''

    virtualenv new-env

'''

## 6. Проверка

Создаем файл test.py:

'''

    #test.py
    def application(env, start_response):
        start_response('200 OK', [('Content-Type','text/html')])
        return [b"Hello World"] # python3
        #return ["Hello World"] # python2

'''

Запускаем uWSGI:

'''

    uwsgi --http :8000 --wsgi-file test.py

'''

В браузере переходим по адресу yourserver.com:8000.
Видим: «Hello, world», значит, мы все сделали правильно и следующие компоненты работают:

'''

Пользователь <-> uWSGI <-> test.py

'''

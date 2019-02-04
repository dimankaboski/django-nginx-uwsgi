# django-nginx-uwsgi

OS: Ubuntu 16.04x86_64
files:
---   'mysite_nginx.conf': simple nginx config
---   'mysite_uwsgi.ini': simple uwsgi config
---   'uwsgi_params': uwsgi params



main post: 'https://habr.com/ru/post/226419/'

--- проверка портов: sudo lsof -nP -i | grep LISTEN


1. Remove apache2 'https://askubuntu.com/questions/176964/permanently-removing-apache2'
    service apache2 stop
    sudo apt remove apache2.*
    sudo apt-get purge apache2 apache2-utils apache2.2-bin apache2-common
        //or
    sudo apt-get purge apache2 apache2-utils apache2-bin apache2.2-common
    sudo apt-get autoremove


2. Install pip3 'https://askubuntu.com/questions/778052/installing-pip3-for-python3-on-ubuntu-16-04-lts-using-a-proxy'
    sudo apt-get -y install python3-pip
    pip3 install virtualenv


3. Install nginx, wsgi etc
    sudo apt-get install pythonX.Y-dev
    sudo apt-get install nginx
    pip3 install uwsgi


4. Add sudo users 'https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-ubuntu-16-04'
    sudo adduser newuser
    usermod -aG sudo newuser


5. Create virtualenv and clone project
    virtualenv new-env

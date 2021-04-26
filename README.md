# Manual по деплою django проектов 
Эту инстркуцкия создовалось в течении трех долгих дней, моей кровью и потом. Думаю она ещё будет дополнятся.

# Обновления
После создание BM нужно обновить систему 
```
sudo apt update
apt list –upgradable
sudo apt upgrade
```

# Безопасность 
`sudo nano /etc/ssh/sshd_config`
## Добовляем в конце файла конфига
```
AllowUsers ТВОЙ_ПОЛЬЗОВАТЕЛЬ
PermitRootLogin no
```
`sudo systemctl reload ssh.service`

#Python 3.8
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.8
sudo apt install python3-pip
sudo apt install -y python3-venv
```

#Venv
```
mkdir /home/USER/env/
python3 -m venv /home/udoms/env/t_env
```

#Cоздание проекта
```
pip3 install django
sudo apt install python-django-common
sudo apt-get install python-django
django-admin startproject PROJ  
```

#WSGI
```
pip3 install uwsgi 
sudo apt install uwgsi
```
В папке с проектом(там должен быть файл manage.py)
`sudo nano uwsgi.ini`
Для пути файла /home/pomig/prj/lite(lite - папка с проектом):
```
project = lite
base = /home/pomig/prj
#
chdir = %(base)/%(project)
home = /home/pomig/prj/env
#
module = lite.wsgi:application
master = true
processes = 5
#
socket = %(base)/%(project)/%(project).sock
chmod-socket = 666
vacuum = true
```

#Nginx
```
sudo apt install nginx
sudo nano /etc/nginx/sites-available/PROJ_NAME.conf
```
Записываем следующее:
```
# the upstream component nginx needs to connect to
upstream django {
	server unix:///home/pomig/prj/lite/lite.sock;
}
# configuration of the server
server {
    listen      80;
    server_name 130.193.35.110;
    charset     utf-8;
    # max upload size
    client_max_body_size 75M;
    # Django media and static files
    location /media  {
        alias /home/pomig/prj/lite/media;
    }
    #
   	location /static {
		alias /home/pomig/prj/lite/static;
	}
    # Send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /home/pomig/prj/lite/uwsgi_params;
    }
}
```
Создаем новый файл uwsgi_params:
`sudo nano /home/pomig/prj/lite/uwsgi_params`
```
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;
uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;
uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```
`sudo ln -s /etc/nginx/sites-available/PROJ_NAME.conf /etc/nginx/sites-enabled/`
#Статика
`nano /home/pomig/prj/lite/lite/settings.py`
```Python
# самое начало
import os
...
...
#самый конец 
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```	
`python manage.py collectstatic`

`mkdir media/img`
`wget https://upload.wikimedia.org/wikipedia/commons/b/b9/First-google-logo.gif -O media/img/media.gif`
##Рестарним nginx для применения изменений
`sudo /etc/init.d/nginx restar`

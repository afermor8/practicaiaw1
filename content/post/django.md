---
title: "Despliegue de aplicaciones Python (Django)"
date: 2023-01-26T10:51:57+01:00
draft: true
tags: ["IAW"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

------------------------------------------

## Tarea 1: Entorno de desarrollo

Vamos a desplegar la aplicación del tutorial de django. Como entorno de desarrollo tienes dos opciones:

1. Que tu entorno de desarrollo se la máquina bravo de tu entorno de desarrollo. Opción que dará más puntos.

2. Que tu entorno de desarrollo sea una máquina de openstack con el sistema operativo que quieras. Opción que dará menos puntos.

Vamos a configurar tu equipo como entorno de desarrollo para trabajar con la aplicación, para ello:
- Realiza un fork del repositorio de GitHub: https://github.com/josedom24/django_tutorial. 
- Crea un entorno virtual de python3 e instala las dependencias necesarias para que funcione el proyecto.
- Comprueba que vamos a trabajar con una base de datos sqlite. ¿Qué fichero tienes que consultar? ¿Cómo se llama la base de datos que vamos a crear?
- Crea la base de datos. A partir del modelo de datos se crean las tablas de la base de datos.
- Crea un usuario administrador.
- Ejecuta el servidor web de desarrollo y entra en la zona de administración (/admin) para comprobar que los datos se han añadido correctamente.
- Crea dos preguntas, con posibles respuestas.
- Comprueba en el navegador que la aplicación está funcionando, accede a la url /polls.
- Configura el servidor web apache2 con el módulo wsgi para servir la página web. Si utilizas como entorno de desarrollo la máquina bravo, se accederá con el nombre python.tunombre.gonzalonazareno.org. Si tu entorno de desarrollo es una máquina de openstack, elige el nombre con el que acceder y entrega la dirección IP de la máquina.

------------------------------------------

He realizado un fork del repositorio de GitHub: https://github.com/josedom24/django_tutorial y he clonado el fork en mi entorno de desarrollo.

```
git clone git@github.com:afermor8/django_tutorial.git
```

He creado un entorno virtual de python3 llamado django. E instalo requirements (en este caso es el paquete django lo que se instalará).

```
python3 -m venv venv/django
source venv/django/bin/activate
cd django_tutorial
pip install -r requirements.txt
```

Compruebo que la base de datos con la que vamos a trabajar es sqlite. En el repositorio clonado accedemos a django_tutorial y en el fichero settings.py podremos encontrar la siguiente información.

```
…
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
…
```

Vemos que la base de datos se llama db.sqlite3.
Creamos la base de datos con el siguiente comando:

```
python manage.py migrate
```

![migrate](/img/django/p1.png)

Creamos un usuario administrador para la base de datos:

```
python3 manage.py createsuperuser
```

En settings.py tenemos que añadir nuestra IP a ALLOWED_HOSTS.

![](/img/django/p2.png)

Ejecutamos el servidor de desarrollo.

```
python3 manage.py runserver 192.168.122.192:8000
```

![](/img/django/p3.png)

Entramos en la zona de administración con el usuario y contraseña creada anteriormente:

![](/img/django/p4.png)

Creo dos preguntas presionando en ‘Add’ al lado de ‘Questions’.

![](/img/django/p5.png)

![](/img/django/p6.png)

![](/img/django/p7.png)

Comprobamos que podemos acceder en el navegador a las preguntas creadas (/polls).

![](/img/django/p8.png)

![](/img/django/p9.png)

Ahora configuramos el servidor web apache2 con el módulo wsgi para servir la página web. Instalamos apache2 si no lo tuvieramos y el módulo wsgi.

```
sudo apt install libapache2-mod-wsgi-py3
```

El fichero wsgi, que será el punto de acceso a la aplicación se encuentra en django_tutorial/django_tutorial/wsgi.py

Creo un virtualhost que tendrá el siguiente contenido (he creado los alias para que se muestre el contenido estático de la aplicación):

```
<VirtualHost *:80>
	ServerName django.arantxa.org

	ServerAdmin webmaster@localhost
	DocumentRoot /home/usuario1/django_tutorial

         Alias /static/admin/ /home/usuario1/venv/django/lib/python3.9/site-packages/django/contrib/admin/static/admin/

        <Directory /home/usuario1/venv/django/lib/python3.9/site-packages/django/contrib/admin/static/admin/>
                Require all granted
        </Directory>
        Alias /static/ /home/usuario1/django_tutorial/polls/static/
        <Directory /home/usuario1/django_tutorial/polls/static>
                Require all granted
        </Directory>

        WSGIDaemonProcess django-tutorial python-path=/home/usuario1/django_tutorial:/home/usuario1/venv/django/lib/python3.9/site-packages
        WSGIProcessGroup django-tutorial
        WSGIScriptAlias / /home/usuario1/django_tutorial/django_tutorial/wsgi.py process-group=django-tutorial
        <Directory /home/usuario1/django_tutorial/django_tutorial>
                Require all granted
        </Directory>

	ErrorLog ${APACHE_LOG_DIR}/error-django-arantxa-org.log
	CustomLog ${APACHE_LOG_DIR}/access-django-arantxa-org.log combined

</VirtualHost>
```

He llamado a la web django.arantxa.org. Creo el enlace del virtualhost y reinicio el servicio.

```
sudo a2ensite django.arantxa.org.conf
sudo systemctl restart apache2
```

Añado a /etc/hosts la resolución estática:

```
192.168.122.192 		django.arantxa.org
```

Por último añadir django.arantxa.org a settings.py (ALLOWED_HOSTS) y cambiar el usuario y grupo a www-data.

![](/img/django/p10.png)

```
cd
chown -R www-data:www-data django_tutorial
```

![](/img/django/p11.png)
![](/img/django/p12.png)
![](/img/django/p13.png)
![](/img/django/p14.png)

------------------------------------------

## Tarea 2: Entorno de producción

Vamos a realizar el despliegue de nuestra aplicación en un entorno de producción, para ello vamos a utilizar nuestro VPS, sigue los siguientes pasos:
- Clona el repositorio en el VPS. 
- Crea un entorno virtual e instala las dependencias de tu aplicación. 
- Instala el módulo que permite que python trabaje con mysql:

```
        (env)$ pip install mysqlclient
```

- Crea una base de datos y un usuario en mysql. 
- Configura la aplicación para trabajar con mysql, para ello modifica la configuración de la base de datos en el archivo settings.py:

```
        DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.mysql',
                'NAME': 'myproject',
                'USER': 'myprojectuser',
                'PASSWORD': 'password',
                'HOST': 'localhost',
                'PORT': '',
            }
        }
```

- Crea una copia de seguridad de la base de datos. Ten en cuenta que en el entorno de desarrollo vas a tener una base de datos sqlite, y en el entorno de producción una mariadb, por lo tanto es recomendable para hacer la copia de seguridad y recuperarla con los comandos: python manage.py dumpdata y python manage.py loaddata, para más información.
- Configura el servidor de aplicaciones uwsgi, creando una unidad de systemd como hicimos en el taller2) y configura nginx como proxy inverso para servir la aplicación.
- Debes asegurarte que el contenido estático se está sirviendo: ¿Se muestra la imagen de fondo de la aplicación? ¿Se ve de forma adecuada la hoja de estilo de la zona de administración?
- Desactiva en la configuración el modo debug a False. Para que los errores de ejecución no den información sensible de la aplicación.
- La página web debe ser accesible usando https, en la URL: https://python.tudominio.algo
- Muestra la página funcionando. En la zona de administración se debe ver de forma adecuada la hoja de estilo.

En este momento, muestra al profesor la aplicación funcionando. Entrega una documentación resumida donde expliques los pasos fundamentales para realizar esta tarea y pantallazos donde se vea que todo está funcionando. (3,5 puntos)

----------------------------------------------

Entro en mi vps y clono el repositorio django_tutorial.

```
ssh arantxa@217.160.225.205
git clone git@github.com:afermor8/django_tutorial.git
```

Creo el entorno virtual e instalo las dependencias:

```
python3 – venv venv/django
source venv/django/bin/activate
pip install -r django_tutorial/requirementes.txt
```

Creo una base de datos y un usuario en mysql (se instalan las dependencias si no las tenemos).

```
sudo apt install python3-dev default-libmysqlclient-dev build-essential
pip install mysqlclient
sudo mysql -u root -p
create database djangodb;
grant all on djangodb.* to 'arantxa'@'%' identified by '*********' with grant option;
```

Modifico el fichero django_tutorial/django_tutorial/settings.py

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'djangodb',
        'USER': 'arantxa',
        'PASSWORD': '*********',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

Creo la migración y creo un superusuario.

```
python3 django_tutorial/manage.py migrate
python3 django_tutorial/manage.py createsuperuser
```

Instalo uwsgi.

```
pip install uwsgi
```

Creo el fichero .ini

```
sudo nano venv/django/django.ini

[uwsgi]
http = :8080
chdir = /home/arantxa/django_tutorial
wsgi-file = /home/arantxa/django_tutorial/django_tutorial/wsgi.py
processes = 4
threads = 2
```

Creo el fichero systemd:

```
sudo nano /etc/systemd/system/uwsgi-django.service

[Unit]
Description=uwsgi-django
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
User=www-data
Group=www-data
Restart=always

ExecStart=/home/arantxa/django/bin/uwsgi /home/aranxa/django/django.ini
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

WorkingDirectory=/home/arantxa/django_tutorial
Environment=PYTHONPATH='/home/arantxa/django_tutorial:/home/arantxa/django/lib/python3.9/site-packages'

PrivateTmp=true
```

Activamos la unidad systemd y la iniciamos:

```
sudo systemctl enable uwsgi-django.service
sudo systemctl start uwsgi-django.service
```

Creo el virtualhost de nginx.

```
sudo nano /etc/nginx/sites-available/django.afm-tars.es

server {
	listen 80;
	listen [::]:80;

	# SSL configuration
	#
	# listen 443 ssl default_server;
	# listen [::]:443 ssl default_server;
	#

	root /home/arantxa/django_tutorial;

	index index.html index.htm index.nginx-debian.html;

	server_name python.afm-tars.es;

	location / {
		proxy_pass http://localhost:8080;
		include proxy_params;
	}
	
	location /static/ {
		alias /home/arantxa/django_tutorial/polls/static/;
	}
	location /static/admin/ {
		alias /home/arantxa/venv/django/lib/python3.9/site-packages/django/contrib/admin/static/admin/;
	}
}
```

Creo el enlace simbólico:

```
sudo ln -s /etc/nginx/sites-available/django.afm-tars.es /etc/nginx/sites-enabled/
```

En settings he puesto DEBUG igual a False.

```
sudo nano django_tutorial/django_tutorial/settings.py 

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = False

ALLOWED_HOSTS = ['python.afm-tars.es','217.160.225.205']
```

He copiado la información de los datos de la aplicación del entorno de desarrollo para cargarlos en el servidor python.afm-tars.es.
Primero en el entorno de desarrollo:

```
python3 django_tutorial/manage.py dumpdata > db.json
scp db.json arantxa@217.160.225.205:.
```

Y en el servidor vps:

```
pip install  pymysql
python3 django_tutorial/manage.py loaddata db.json 
```

Creo la resolución estática en /etc/hosts.

```
sudo nano /etc/hosts

217.160.225.205         python.afm-tars.es
```

En mi vps he creado una directiva CNAME para que python.afm-tars.es vaya a walle.afm-tars.es

![](/img/django/p15.png)

Reinicio el servicio y ya debería funcionar la web.

```
sudo systemctl restart nginx
```

![](/img/django/p16.png)
![](/img/django/p17.png)
![](/img/django/p18.png)
![](/img/django/p19.png)
![](/img/django/p20.png)

Para que la web sea accesible desde la URL https://python.afm-tar.es con https primero tenemos que tener el certificado de letsencrypt. Si no lo tuviéramos habría que seguir los siguientes pasos:

```
sudo apt install certbot
sudo apt install python3-certbot-nginx
sudo certbot --nginx

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: python.afm-tars.es
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
Requesting a certificate for python.afm-tars.es
Performing the following challenges:
http-01 challenge for python.afm-tars.es
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/django.afm-tars.es
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/django.afm-tars.es

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://python.afm-tars.es
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/python.afm-tars.es/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/python.afm-tars.es/privkey.pem
   Your certificate will expire on 2023-04-24. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again with the "certonly" option. To non-interactively
   renew *all* of your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Cuando tengamos el certificado modificamos el virtualhost de nginx dejándolo de la siguiente forma:

```
sudo nano /etc/nginx/sites-available/django.afm-tars.es

server {
	listen 80;
	listen [::]:80;
	server_name python.afm-tars.es;
	return 301 https://$server_name$request_uri;
}

server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name python.afm-tars.es;
	ssl_certificate /etc/letsencrypt/live/python.afm-tars.es/fullchain.pem; # managed by Certbot
	ssl_certificate_key /etc/letsencrypt/live/python.afm-tars.es/privkey.pem; # managed by Certbot
	include /etc/letsencrypt/options-ssl-nginx.conf;
	ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

	root /home/arantxa/django_tutorial;

	index index.php;

	location / {
		proxy_pass http://localhost:8080;
		include proxy_params;
	}
	
	location /static/ {
		alias /home/arantxa/django_tutorial/polls/static/;
	}
	location /static/admin/ {
		alias /home/arantxa/venv/django/lib/python3.9/site-packages/django/contrib/admin/static/admin/;
	}
}
```

![](/img/django/p21.png)
![](/img/django/p22.png)
![](/img/django/p23.png)
![](/img/django/p24.png)

------------------------------------------

## Tarea 3: Modificación de nuestra aplicación

Vamos a realizar cambios en el entorno de desarrollo y posteriormente vamos a subirlas a producción. Vamos a realizar tres modificaciones, pero recuerda que primero lo haces en el entorno de desarrollo, y luego tendrás que llevar los cambios a producción:

1. Modifica la página inicial donde se ven las encuestas para que aparezca tu nombre: Para ello modifica el archivo django_tutorial/polls/templates/polls/index.html.
2. Modifica la imagen de fondo que se ve la aplicación.
3. Vamos a crear una nueva tabla en la base de datos, para ello sigue los siguientes pasos:

- Añade un nuevo modelo al fichero polls/models.py:

```
            class Categoria(models.Model):        
                  Abr = models.CharField(max_length=4)
                  Nombre = models.CharField(max_length=50)
          
                  def __str__(self):
                          return self.Abr+" - "+self.Nombre
```

- Crea una nueva migración.
- Y realiza la migración. 
- Añade el nuevo modelo al sitio de administración de django:
Para ello cambia la siguiente línea en el fichero polls/admin.py:

```
            from .models import Choice, Question
```

Por esta otra:

```
            from .models import Choice, Question, Categoria
```

Y añade al final la siguiente línea:

```
            admin.site.register(Categoria)
```

- Despliega el cambio producido al crear la nueva tabla en el entorno de producción.

Explica los cambios que has realizado en el entorno de desarrollo y cómo lo has desplegado en producción para cada una de las modificaciones. Entrega pantallazos donde se vean las distintas modificaciones y que todo está funcionando. (3,5 puntos)

-----------------------------------------------


Modificamos la página inicial en el servidor de desarrollo y añado mi nombre al html de la siguiente forma:

```
cd django_tutorial/polls/templates/polls/
sudo nano index.html
```

```
{% load static %}

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">

<h1>Arantxa Fernández Morató</h1>
<h2>Práctica Despliegues de Aplicaciones Python (Django). ASIR 2</h2>

{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
    <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

![](/img/django/p25.png)

Ahora cambiamos la imagen de fondo, la cual se encuentra en `django_tutorial/polls/static/polls/images/background.jpg`. Me descargo otra imagen y le doy el nombre de la actual para que la sobreescriba.

```
cd django_tutorial/polls/static/polls/images/
sudo wget https://img.freepik.com/vector-gratis/fondo-ondulado-abstracto-marron-claro_1108-592.jpg?w=2000 -O background.jpg
```

Comprobamos los cambios en la web.

![](/img/django/p26.png)


Creo una nueva tabla en la base de datos. Para ello añadimos al final del fichero models.py lo siguiente:

```
sudo nano django_tutorial/polls/models.py

class Categoria(models.Model):        
    Abr = models.CharField(max_length=4)
    Nombre = models.CharField(max_length=50)

    def __str__(self):
        return self.Abr+" - "+self.Nombre
```

Para realizar los siguientes pasos he tenido que volver a cambiar el usuario y grupo propietario de django_tutorial: `sudo chown -R usuario:usuario django_tutorial/`

Creo una nueva migración:

```
python3 django_tutorial/manage.py makemigrations
```

Realizo la migración:

```
python3 django_tutorial/manage.py migrate
```

![](/img/django/p27.png)


Añado un nuevo modelo al sitio de administración de django. Para hacer esto, en el fichero admin.py, añado a la línea `from .models import Choice, Question` la nueva tabla llamada Categoría. Y añadimos al final la línea `admin.site.register(Categoria)`.

```
sudo nano django_tutorial/polls/admin.py
```

```
from .models import Choice, Question, Categoria
...
admin.site.register(Categoria)
```

![](/img/django/p28.png)


Comprobamos que funciona la web en el entorno de desarrollo. Para ello tenemos que volver a cambiar el usuario y grupo de django_tutorial: `sudo chown -R www-data:www-data django_tutorial/`

![](/img/django/p29.png)

![](/img/django/p30.png)


Copio lainformación de los datos de la aplicación en el entorno de desarrollo en nuestro servidor de producción.

En el entorno de desarrollo:

```
python3 django_tutorial/manage.py dumpdata > db2.json
scp db.json arantxa@217.160.225.205:.
```

En el servidor de producción, antes de actualizar los datos tenemos que actualizar el repositorio git (git pull).

> Al actualizar el repositorio, el fichero settings.py se actualiza con el host que teníamos en el entorno de desarrollo. Debemos asegurarnos de que en el entorno de producción el host permitido es el siguiente: `ALLOWED_HOSTS = ['python.afm-tars.es','217.160.225.205']`

Una vez actualizado el repositorio en el entorno de producción realizamos la migración:

```
python3 django_tutorial/manage.py migrate
```

Ya podremos actualizar los datos:

```
python3 django_tutorial/manage.py loaddata db.json
```

Reiniciamos el servicio uwsgi-django:

```
sudo systemctl restart uwsgi-django
```

Comprobamos que los cambios (la tabla categoría, nuestro nombre y el fondo de pantalla) se han aplicado correctamente en el entorno de producción:

![](/img/django/p31.png)

![](/img/django/p32.png)

![](/img/django/p33.png)

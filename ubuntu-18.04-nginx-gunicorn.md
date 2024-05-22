# You may need to edit the source list in the following file.
#sudo nano /etc/apt/sources.list

# add user 

sudo useradd -g www-data myuser

# set the password for myuser user 

sudo passwd myuser

# Installing the packages from the Ubuntu repositories

sudo apt update

sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl

sudo apt install gdal-bin python-gdal python3-gdal


# Creating the PostgreSQL database and user
sudo -u postgres psql

CREATE DATABASE project;

CREATE USER myuser WITH PASSWORD 'password';

ALTER ROLE myuser SET client_encoding TO 'utf8';

ALTER ROLE myuser SET default_transaction_isolation TO 'read committed';

ALTER ROLE myuser SET timezone TO 'UTC';

GRANT ALL PRIVILEGES ON DATABASE project TO myuser;

\q

# Creating a python Virtual Environment for your Project

sudo mkdir /home/myuser

mkdir /home/myuser/project

cd /home/myuser/project

python3 -m venv myprojectenv

source myprojectenv/bin/activate

pip install django gunicorn psycopg2-binary

pip install djangorestframework

pip install djangorestframework-gis

pip install django-leaflet

django-admin startproject project /home/myuser/project

# Add allowed host in settings.py file inside your django project
sudo nano /home/myuser/project/project/settings.py

ALLOWED_HOSTS = ['your_server_domain_or_IP', 'second_domain_or_IP', . . ., 'localhost']

# Then change database settings inside setting file

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'project',
        'USER': 'myuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
}

# Set STATIC_ROOT var

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Add GIS apps
INSTALLED_APPS = [
    ...
    'rest_framework',
    'rest_framework_gis',
    'leaflet',
    ]

import os

STATIC_ROOT = os.path.join(BASE_DIR, 'static/')


# Migrate database

/home/myuser/project/manage.py makemigrations

/home/myuser/project/manage.py migrate


# Create superuser

/home/myuser/project/manage.py createsuperuser

# Collect static

/home/myuser/project/manage.py collectstatic


# Add folder permissions to user

chown myuser:www-data /home/myuser/ -R

sudo chmod 775 /home/myuser -R


# Creating systemd Socket and Service Files for Gunicorn
sudo nano /etc/systemd/system/gunicorn.service


# add following to the file:

[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=myuser
Group=www-data
WorkingDirectory=/home/myuser/project
ExecStart=/home/myuser/project/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/myuser/project/project.sock project.wsgi:application

[Install]
WantedBy=multi-user.target


# Create Symlink
sudo systemctl enable gunicorn.service


# Start Gunicorn using systemctl
sudo systemctl start gunicorn


# Nginx config

sudo nano /etc/nginx/sites-available/project



# Add the following to the file:

server {

    listen 80;
    
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    
    location /static/ {
    
        root /home/myuser/project;
    }

    location / {

        include proxy_params;
        
        proxy_pass http://unix:/home/myuser/project/project.sock;
    }
    
}

# Link directory

sudo ln -s /etc/nginx/sites-available/project /etc/nginx/sites-enabled

# Nginx restart

sudo systemctl restart nginx

# Nginx full

sudo ufw allow 'Nginx Full'




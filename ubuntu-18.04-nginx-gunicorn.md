# You may need to edit the source list in the following file.
#sudo nano /etc/apt/sources.list

# Installing the packages from the Ubuntu repositories
sudo apt update

sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl

# Creating the PostgreSQL database and user
sudo -u postgres psql
CREATE DATABASE project;

CREATE USER myuser WITH PASSWORD 'password';

ALTER ROLE myprojectuser SET client_encoding TO 'utf8';

ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';

ALTER ROLE myuser SET timezone TO 'UTC';

GRANT ALL PRIVILEGES ON DATABASE project TO myuser;

\q

# Creating a python Virtual Environment for your Project
mkdir ~/project

cd ~/project

python3 -m venv myprojectenv

source projectenv/bin/activate

django-admin startproject project ~/project

# Add allowed host in settings.py file inside your django project
nano ~/project/project/settings.py

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

import os

STATIC_ROOT = os.path.join(BASE_DIR, 'static/')


# Migrate database

~/project/manage.py makemigrations

~/project/manage.py migrate


# Create superuser

~/project/manage.py createsuperuser

# Collect static

~/myprojectdir/manage.py collectstatic


# Creating systemd Socket and Service Files for Gunicorn





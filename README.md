# Video is comming soon...
# deploy-django-app-postgresql-docker

![architecture diagram](deploy.png)

Demo project to deploy to bare metal server (or any system) using docker, docker-compose and certbot to automatically and for free obtain ssl certificate.


# Todo

1. Create docker-compose.yaml
1. Create templates for all the services

## Create Django-service

1. Create `django-backend` folder for relaited service
1. Create the description of django service
1. Create `Dockerfile` in `django-backend`
1. Create virtual env in `django-backend`
    ```bash
    python3 -m venv env  # create env
    . ./env/bin/activate # activate env
    ```
1. Start new django project in `django-backend`
    ```bash
    pip install -U Django # install or update Django
    pip install -U gunicorn # install Gunicorn webserver
    django-admin startproject proj # create new django project
    mv ./proj ./src # rename outer proj folder to src (just for convenience)
    ```
1. Create `requirements.txt` in `django-backend` folder
    ```bash
    pip freeze > ./requirements.txt
    ```
1. Ajust Dockerfile to build the image with django
1. Try to run the service

    ```bash
    docker-compose up
    ```
    
    ```bash
    django-backend_1  | [2021-08-30 08:30:20 +0000] [7] [INFO] Starting gunicorn 20.1.0
    django-backend_1  | [2021-08-30 08:30:20 +0000] [7] [INFO] Listening at: http://0.0.0.0:8000 (7)
    django-backend_1  | [2021-08-30 08:30:20 +0000] [7] [INFO] Using worker: sync
    django-backend_1  | [2021-08-30 08:30:20 +0000] [8] [INFO] Booting worker with pid: 8
    django-backend_1  | [2021-08-30 08:30:20 +0000] [9] [INFO] Booting worker with pid: 9
    django-backend_1  | [2021-08-30 08:30:20 +0000] [10] [INFO] Booting worker with pid: 10
    ```
## Create DB-service

1. Ajust `docker-compose.yml` file
1. Add `psycopg2-binary` to django requirements
    ```
    pip install -U psycopg2-binary
    pip freeze > ./requirements.txt
    ```
1. Ajust django `settings.py` to use db-service as following:

    ```python
    # settings.py
    # edit the following block
    # exposing your passwords to github is not a good idea! Use import and gitignore
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': 'deploy-db', # as a POSTGRES_DB
            'USER': 'deploy-test-user', # as a POSTGRES_USER
            'PASSWORD': 'deploy-test-password', # as a POSTGRES_PASSWORD
            'HOST': 'postgresql-db', # as the DB's service name in docker-compose.yml
            'PORT': '', # default
        }
    }
    ```
1. Try to run both services:

    ```bash
    postgresql-db_1   | PostgreSQL init process complete; ready for start up.
    postgresql-db_1   | 
    postgresql-db_1   | 2021-08-30 10:38:43.954 UTC [1] LOG:  starting PostgreSQL 13.2 (Debian 13.2-1.pgdg100+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit
    postgresql-db_1   | 2021-08-30 10:38:43.954 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
    postgresql-db_1   | 2021-08-30 10:38:43.954 UTC [1] LOG:  listening on IPv6 address "::", port 5432
    postgresql-db_1   | 2021-08-30 10:38:43.956 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
    postgresql-db_1   | 2021-08-30 10:38:43.959 UTC [75] LOG:  database system was shut down at 2021-08-30 10:38:43 UTC
    postgresql-db_1   | 2021-08-30 10:38:43.962 UTC [1] LOG:  database system is ready to accept connections
    ```
    try to migrate: connect to django container and run:

    ```sh
    python manage.py migrate
    Operations to perform:
    Apply all migrations: admin, auth, contenttypes, sessions
    Running migrations:
    Applying contenttypes.0001_initial... OK
    Applying auth.0001_initial... OK
    Applying admin.0001_initial... OK
    Applying admin.0002_logentry_remove_auto_add... OK
    Applying admin.0003_logentry_add_action_flag_choices... OK
    Applying contenttypes.0002_remove_content_type_name... OK
    Applying auth.0002_alter_permission_name_max_length... OK
    Applying auth.0003_alter_user_email_max_length... OK
    Applying auth.0004_alter_user_username_opts... OK
    Applying auth.0005_alter_user_last_login_null... OK
    Applying auth.0006_require_contenttypes_0002... OK
    Applying auth.0007_alter_validators_add_error_messages... OK
    Applying auth.0008_alter_user_username_max_length... OK
    Applying auth.0009_alter_user_last_name_max_length... OK
    Applying auth.0010_alter_group_name_max_length... OK
    Applying auth.0011_update_proxy_permissions... OK
    Applying auth.0012_alter_user_first_name_max_length... OK
    Applying sessions.0001_initial... OK
    ```

## Create NGINX-service

1. Create folder `nginx`
1. Create `Dockerfile` in it
    ```docker
    FROM nginx
    # next line copies nginx configuration to the proxy-server
    COPY ./default.conf /etc/nginx/conf.d/default.conf 
    ```
1. Create `default.conf` with `nginx` configurarion

    ```nginx
    upstream innerdjango {
        server django-backend:8000;
        # connection to the inner django-backend service
        # here `django-backend` is the service's name in
        # docker-compose.yml, it is resolved by docker to inner IP address.
        # The `innerdjango` is just te name of upstream, used by nginx below. 
    }
    server {
        # the connection to the outside world
        # will be changed to incorporate cert's bot and ssl
        # just to test it localy for now
        listen 80; # port exposed to outside world. Needs to be opened in docker-compose.yml
        # server_name example.com;
        location / {
            # where to redirect `/` requests
            # to inner `innerdjango` upstream
            proxy_pass http://innerdjango;
        }
    }
    ```
1. Test what's been done so far:
    
    1. Run docker-compose
        ```bash
        docker-compose up --build
        ```
    1. Navigate to [127.0.0.1](http://127.0.0.1) in your browser.
    You must see something like this
        ```
        Invalid HTTP_HOST header: 'innerdjango'. You may need to add 'innerdjango' to ALLOWED_HOSTS.
        ```
    It's ok for now.
    If you see it everything works...
# To be continued...
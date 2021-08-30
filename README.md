# Video is comming soon...
# deploy-django-app-postgresql-docker

![architecture diagram](deploy.png)

Demo project to deploy to bare metal server (or any system) using docker, docker-compose and certbot to automatically and for free obtain ssl certificate.


# Todo

1. Create docker-compose.yaml
1. Create templates for all the services
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
# To be continued...
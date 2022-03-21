# cookiecutter_setup 
Taken from JustDjango at https://www.youtube.com/watch?v=8c14GBrbglw&t=154s

### Please note, usernames and passwords in currentProject/.envs/.local/.postgres must be changed by generating your own CookieCutter build from line 37. As these usernames and passwords are now compromised at publishing

## Within VSCode - Install CookieCutter, Virtual Environment, Source
```
pip install cookiecutter
pip install virtualenv
pip install source
pip install psycopg2-binary 
```
### Within VSCode - make current project directory
```mkdir currentProject
cd currentProject
```
### Within VSCode - Create virtual environment and activate 
#### (WINDOWS)
```virtualenv env
env/Scripts/Activate.ps1
```
#### (LINUX)
```virtualenv env
source env/bin/activate
```
### Within VSCode - Install Django into the virtual environment
`pip install django`

### Within Terminal - Django-Admin Start
`django-admin startproject currentProject .`

### Within Terminal - Initialize and RunServer(development)
```
python manage.py migrate
python manage.py runserver
```
### Within Browser - Test to make sure website is running at http://127.0.0.1:8000/ 

## Cookiecutter Steps 
#### https://cookiecutter-django.readthedocs.io/en/latest/developing-locally.html

### Create a virtualenv:

`python3.9 -m venv <virtual env path>`

### Activate the virtualenv you have just created:
#### WINDOWS
`advancedEnv/Scripts/Activate.ps1`
#### LINUX
`source <virtual env path>/bin/activate`

### Install cookiecutter-django:

`cookiecutter gh:cookiecutter/cookiecutter-django`

### Install development requirements:
```
cd <what you have entered as the project_slug at setup stage>
pip install -r requirements/local.txt
git init # A git repo is required for pre-commit to install
pre-commit install
```
### Create a new PostgreSQL database using createdb:

`createdb --username=postgres <project_slug>`

### Set the environment variables for your database(s):

`export DATABASE_URL=postgres://postgres:<password>@127.0.0.1:5432/<DB name given to createdb>`

# Optional: set broker URL if using Celery
`export CELERY_BROKER_URL=redis://localhost:6379/0`

### Deactivate the virtual AdvancedEnv
`deactivate`

### Transfer files over
* .envs Folder
* compose Folder
* docs Folder
* .dockerignore
* .gitignore
* local.yaml
* production.yaml


### Configure Dockerfile
open currentProject/compose/local/django/Dockerfile
#### Simplify requirements instead of broken down requirements per stage
change Line 19: COPY ./requirements . 
to: COPY ./requirements.txt .

### Activate original virtual environment
`cd currentProject`

#### (WINDOWS)
```virtualenv env
env/Scripts/Activate.ps1
```
#### (LINUX)
```virtualenv env
source env/bin/activate
```

### Freeze Requirements
`pip freeze > requirements.txt`

### Add additional lines to requirements.txt as created in the generated cookiecutter_django project my_awesome_project>config>settings>base.py

Add the line: `psycopg2==2.9.1`
Add the line: `django-environ==0.4.5`

#### Add environ from my_awesome_project>config>settings>settings.py to currentProject settings.py
Line 6: (at the beginning of the file)
`import environ`

Lines 13-16 (after base directory):
```
env = environ.Env()
READ_DOT_ENV_FILE = env.bool("DJANGO_READ_DOT_ENV_FILE", default=False)
if READ_DOT_ENV_FILE:
    # OS environment variables take precedence over variables from .env
    env.read_env(str(ROOT_DIR / ".env"))
```
Replace env.read_env(str(ROOT_DIR / ".env")) with env.read_env(str(BASE_DIR / ".env"))

Lines 43 & 44: (replace lines 82-87 in currentProject/settings.py)
```
DATABASES = {"default": env.db("DATABASE_URL")}
DATABASES["default"]["ATOMIC_REQUESTS"] = True
```
### Change requirements variable code for build environment to standard requirements and remove docs lines 38-53
FROM: `-r ${BUILD_ENVIRONMENT}.txt`
TO: `-r requirements.txt`
Remove docs service:  lines 38-53

### Change Start command in currentProject/compose/local/django/start
FROM: Line (9) `python manage.py runserver_plus 0.0.0.0:8000`
TO: Line (9) `python manage.py runserver 0.0.0.0:8000`
DO NOT FORGET TO CONFIGURE THE IP Address And Port!!!

### Specify Allowed Hosts in currentProject > currentProject > settings.py
Can copy pre-made settings located in generated project>config>settings>local.py

`ALLOWED_HOSTS = ["localhost", "0.0.0.0", "127.0.0.1"]`

### Build the docker file
`docker-compose -f local.yml build`

### Run images as containers
`docker-compose -f local.yml up`
If an error appears due to an existing container name, rename the yml service

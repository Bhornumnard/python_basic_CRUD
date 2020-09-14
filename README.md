# basic python CRUD

## reference knowledge
[reference link](https://bezkoder.com/django-crud-mysql-rest-framework/)
![project structure](https://bezkoder.com/wp-content/uploads/2020/03/django-mysql-crud-rest-framework-example-project-structure.png)

## command for create this project
### 1 install Django
```
py -m pip install Django
```
### 2 create project 
```
django-admin startproject "projectname"
```
### 3 install mysql client
```
pip install mysqlclient
```
### 4 install djangorestframework
```
pip install djangorestframework
```
### 5 install cors
```
pip install django-cors-headers
```
### 6 create project
```
django-admin startproject "project name"
```
### 7 create app
```
py manage.py startapp "app name"
```
### 8 setting in setting.py

> ### use app in project
```python
INSTALLED_APPS = [ 
    ... 
    # Tutorials application 
    'tutorials.apps.TutorialsConfig',
    # Django REST framework 
    'rest_framework',
    # CORS
    'corsheaders',
```

> ### use CORS in middleware
```python
#add this code before middleware
CORS_ORIGIN_ALLOW_ALL = False
CORS_ORIGIN_WHITELIST = (
    'http://localhost:8080',
)

MIDDLEWARE = [ 
    ... 
    **# CORS** 
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
]
```

> ### connecting mysql database
```python
DATABASES = { 
    ... 
    'default' : {
        # setting database to use (mysql)
        'ENGINE': 'django.db.backends.mysql',
        # database name
        'NAME': 'Tutorials',
        # username in mysql workbench
        'USER': 'root',
        # password in mysql workbench
        'PASSWORD': 'tak5936609',
        # localhost or 127.0.0.1
        'HOST': 'localhost',
        # mysql port is 3306
        'PORT': '3306',
    }
}
```

### create models at models.py
```python
from django.db import models


class Tutorial(models.Model):
    title = models.CharField(max_length=70, blank=False, default='')
    description = models.CharField(max_length=200,blank=False, default='')
    published = models.BooleanField(default=False)
```
### 10 create database in mysql
<li> open mysql 
<li> create database
<li> create database name example Tutorials

### 11 create migration and table
```
# create migration
py manage.py makemigration "app name example tutorials"
# create table
py manage.py migrate 'app name example tutorials' 
```

### 12 setting route and create router url

>#### create urls.py in app(tutorial)
```python
from django.conf.urls import url 
from tutorials import views 

urlpatterns = [ 
    url(r'^api/tutorials$', views.tutorial_list),
    url(r'^api/tutorials/(?P<pk>[0-9]+)$', views.tutorial_detail),
    url(r'^api/tutorials/published$', views.tutorial_list_published)
]
```
> #### setting url of urls.py  in project folder

```python 
from django.contrib import admin
from django.conf.urls import url, include 


urlpatterns = [
    path('admin/', admin.site.urls),
    url(r'^', include('tutorials.urls')),
]
```

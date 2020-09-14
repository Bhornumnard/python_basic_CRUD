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
### 6 create app
```
py manage.py startapp "app name"
```
### 7 setting in setting.py

> #### use app in project
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

### 8 create models at models.py
```python
from django.db import models


class Tutorial(models.Model):
    title = models.CharField(max_length=70, blank=False, default='')
    description = models.CharField(max_length=200,blank=False, default='')
    published = models.BooleanField(default=False)
```
### 9 create database in mysql
<li> open mysql 
<li> create database
<li> create database name example Tutorials

### 10 create migration and table
```
# create migration
py manage.py makemigration "app name example tutorials"
# create table
py manage.py migrate 'app name example tutorials' 
```

### 11 setting route and create router url

>#### create urls.py in app(tutorial)
```python
from django.conf.urls import url 
from tutorials import views 

urlpatterns = [ 
    url(r'^api/tutorials$', 
    # setting path for CRUD
    views.tutorial_list),
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
    # route to tutorials.urls file 
    url(r'^', include('tutorials.urls')),
]
```

### 12 Create Serializer class for Data Model
> #### create serializers.py in tutorials file
```python 
from rest_framework import serializers 
from tutorials.models import Tutorial
 
 
class TutorialSerializer(serializers.ModelSerializer):
    # create class Meta and add 2 attribute (model fields)
    class Meta:
        model = Tutorial
        fields = ('id',
                  'title',
                  'description',
                  'published')
```
### 13 controller data by logic in views.py 

```python
from django.shortcuts import render

# Create your views here.
from django.shortcuts import render

from django.http.response import JsonResponse
from rest_framework.parsers import JSONParser 
from rest_framework import status

from tutorials.models import Tutorial
from tutorials.serializers import TutorialSerializer
from rest_framework.decorators import api_view


@api_view(['GET', 'POST', 'DELETE'])
def tutorial_list(request):
    # GET list of tutorials, POST a new tutorial, DELETE all tutorials
    if request.method == 'GET':
        tutorials = Tutorial.objects.all()
        
        title = request.GET.get('title', None)
        if title is not None:
            tutorials = tutorials.filter(title__icontains=title)
        
        tutorials_serializer = TutorialSerializer(tutorials, many=True)
        return JsonResponse(tutorials_serializer.data, safe=False)
        # 'safe=False' for objects serialization
    elif request.method == 'POST':
        tutorial_data = JSONParser().parse(request)
        tutorial_serializer = TutorialSerializer(data=tutorial_data)
        if tutorial_serializer.is_valid():
            tutorial_serializer.save()
            return JsonResponse(tutorial_serializer.data, status=status.HTTP_201_CREATED) 
        return JsonResponse(tutorial_serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
def tutorial_detail(request, pk):
    # find tutorial by pk (id)
    try: 
        tutorial = Tutorial.objects.get(pk=pk) 
    except Tutorial.DoesNotExist: 
        return JsonResponse({'message': 'The tutorial does not exist'}, status=status.HTTP_404_NOT_FOUND) 

    # GET / PUT / DELETE tutorial
    if request.method == 'GET': 
        tutorial_serializer = TutorialSerializer(tutorial) 
        return JsonResponse(tutorial_serializer.data) 
    elif request.method == 'PUT': 
        tutorial_data = JSONParser().parse(request) 
        tutorial_serializer = TutorialSerializer(tutorial, data=tutorial_data) 
        if tutorial_serializer.is_valid(): 
            tutorial_serializer.save() 
            return JsonResponse(tutorial_serializer.data) 
        return JsonResponse(tutorial_serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    elif request.method == 'DELETE': 
        tutorial.delete() 
        return JsonResponse({'message': 'Tutorial was deleted successfully!'}, status=status.HTTP_204_NO_CONTENT)
    elif request.method == 'DELETE':
        count = Tutorial.objects.all().delete()
        return JsonResponse({'message': '{} Tutorials were deleted successfully!'.format(count[0])}, status=status.HTTP_204_NO_CONTENT)
    
@api_view(['GET'])
def tutorial_list_published(request):
    tutorials = Tutorial.objects.filter(published=True)
        
    if request.method == 'GET': 
        tutorials_serializer = TutorialSerializer(tutorials, many=True)
        return JsonResponse(tutorials_serializer.data, safe=False)
```


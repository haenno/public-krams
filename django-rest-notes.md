# Dajango REST exploration and testing notes

## Setup prcedure

### General setup

Installation path:

```sh
pipenv --python 3.10.4 # or latest current python 3 version on system
pipenv run pip install Django==4.0.3
pipenv run django-admin startproject api_be_proj .
pipenv run django-admin startapp api_be_app
pipenv run pip install djangorestframework==3.13.1
```

Add to `settings.py` in porject folder `api_be_proj`:

```python
INSTALLED_APPS = [
    ...
    'api_be_app',
    'rest_framework',
    'drf_yasg', # for swagger
]


REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly',
    ]
}
```

Maybe also change or expand the `ALLOWED_HOSTS` to your usecase (e.g. the domain your server is running at or `localhost` or `0.0.0.0`).

Change in `urls.py` from

```python
from django.urls import include
```

to

```python
from django.urls import path, include, re_path

```

### Swagger API

Install:

```sh
pipenv run pip install drf-yasg==1.20.0 # for swagger
```

Add in `urls.py` before the `URLPATTERNS`:

```python

# for swagger: start
from rest_framework import permissions
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
schema_view = get_schema_view(
    openapi.Info(
        title="Words API",
        default_version='v1',
        description="Welcome to the wonderful world of words!",
        terms_of_service="https://github.com/haenno/",
        contact=openapi.Contact(email="haenno@web.de"),
        license=openapi.License(name="The Unlicense"),
    ),
    public=True,
    permission_classes=(permissions.AllowAny,),
)
# for swagger: end

```

Then also add in `urls.py`:

```python
urlpatterns = [
    re_path(r'^spec(?P<format>\.json|\.yaml)$',
            schema_view.without_ui(cache_timeout=0), name='schema-json'),  # for swagger
    path('doc/', schema_view.with_ui('swagger', cache_timeout=0),
         name='schema-swagger-ui'),  # for swagger
    path('redoc/', schema_view.with_ui('redoc', cache_timeout=0),
         name='schema-redoc'),  # for swagger

         ...
]
```

### Static files

Create a folder `staticfiles` in the base directory and add to `settings.py`:

```python
import os # for staticfiles

STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles') # for staticfiles
```

### Commands to use

```python
pipenv run python manage.py makemigrations
pipenv run python manage.py migrate
pipenv run python manage.py collectstatic
pipenv run python manage.py runserver 0.0.0.0:5000
```

## Create a new API endpoit

Here with the example of `words`. List and add words over a public API endpoint.

First, create a new folder in the APP directory `api_be_app\words\`. Then create new files in that folder:

A empty `__init__.py`.

A file `serializers.py` with:

```python
from rest_framework import serializers

from django.http import HttpResponse # for index page

from api_be_app.models import Words


class WordsSerializer(serializers.ModelSerializer):
    class Meta:
        model = Words
        fields = '__all__'
```

A file `urls.py` with:

```python
from django.urls import path

from api_be_app.words.views import WordAddAPIView, WordsListAPIView

app_name = 'api_be_app'

urlpatterns = [
    path('', lambda request: HttpResponse('<html><body><p><b>Proceed to</b> \
        <a href="doc/">doc</a>, <a href="redoc/">redoc</a>, <a href="spec.json">spec.json</a> \
        or <a href="spec.yaml">spec.yaml</a>.</p></body></html>'), name='index'), # for index page
    path('list-words/', WordsListAPIView.as_view(), name='list-words'),
    path('add-word/', WordAddAPIView.as_view(), name='add-word'),
]
```

A file `views.py` with:

```python
from django.http import HttpResponse

from rest_framework.generics import ListAPIView, CreateAPIView
from rest_framework.response import Response

from api_be_app.models import Words
from api_be_app.words.serializers import WordsSerializer


class WordsListAPIView(ListAPIView):
    model = Words
    serializer_class = WordsSerializer
    queryset = Words.objects.all().order_by('word')

class WordAddAPIView(CreateAPIView):
    model = Words
    serializer_class = WordsSerializer
    queryset = Words.objects.all()

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

Then some changes in the files in the APP directory `api_be_app`.

In the `admin.py` add:

```python
from api_be_app.models import Words
admin.site.register(Words)
```

In the `models.py` add:

```python
class Words(models.Model):
    word = models.CharField(max_length=50, unique=True)
```

Finally add to the `urls.py` in the project folder `api_be_proj`:

```python
urlpatterns = [
    ...
    path('words/v1/', include('api_be_app.words.urls')),
]
```


## Links

- API documentation <https://django-rest-framework-old-docs.readthedocs.io/en/3.7.7/topics/documenting-your-api/#documenting-your-api>
- Swagger for Django <https://www.jasonmars.org/2020/04/22/add-swagger-to-django-rest-api-quickly-4-mins-without-hiccups/>

## Deprecated / for later removal

### Rest framework API docs

Add to `urls.py` from base dir for automatic API documentation:

```python
from rest_framework.documentation import include_docs_urls

urlpatterns = [
    ...
    path('docs/', include_docs_urls(title='My API title',
                                    authentication_classes=[],
                                    permission_classes=[])),
]

# not yet, later: pipenv run pip install django-environ==0.8.1
# not any more: pipenv run pip install coreapi==2.3.3 # seems depricated, was neccesary for /docs/ url

```

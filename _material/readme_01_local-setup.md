# Djangoku - Local Setup

This app is meant to be a bootstrap example of how to create a django project with a postgres database and deploy it to heroku.

## Dependencies
* [Python 3](http://www.python.org) (preferrably 3.6+)
* [Postgres 11](https://postgresapp.com/)

## Setup

### Basic Setup

Those steps are the basic steps that usually apply to any Python project.

* Create a git repo: `git init`
* Create a virtualenvironment: `python3.7 -m venv virtualenv_alpha`
* Add `virtualenv_alpha/` folder to `.gitignore`
* Activate virtualenv: `source virtualenv_alpha/bin/activate`

### Django and PostGres preparation

Specific steps for Django applications with a postgres database.

* Install Django: `pip install django`
* Start Django project: `django-admin.py startproject djangoku .` (note the `.`at the end; it created the django project in the current directory.)
* Add `db.sqlite3`to `.gitignore`
* Create settings for multiple environments (starting with local):
    * Create a new settings folder: `mkdir djangoku/settings`
    * Move and rename `settings.py`: `mv djangoku/settings.py djangoku/settings/base.py`
    * Create settings for your local environment: `touch djangoku/settings/__init__.py && echo "from .base import *" >> djangoku/settings/local_alpha.py`
* Start an app for your Django project: `python manage.py startapp pages`
* Add `'pages',` app to `INSTALLED_APPS` list in `settings/base.py`
* Install `dj-database-url`: `pip install dj-database-url
* Install `psycopg2`: `pip install psycopg2
* Create a new database:
    * Open postgres shell: `psql`
    * Create database: `CREATE DATABASE djangoku_local;` (Note the `;` at the end)
    * Quit postgres shell: `\q`

### Setup your environment

Important data should not live in your code or be part of the version control system. That’s what environment variables are for.

* Create an `.env` file: `touch .env`
* Add `.env` to `.gitignore`
* Get `python-dotenv` to load env variables conveniently: `pip install python-dotenv`
* Create a new secret key (because the old one might have been added to git in the meantime):
    * Run the django shell in the terminal: `python manage.py shell`
    * Import the helper function: `from django.core.management.utils import get_random_secret_key`
    * Get a new secret key: `get_random_secret_key()`
    * Copy the output (e.g. `'NEWLY%G3GENERATEDsecret_KEY!'`) including `'` and save it for later (see below)

#### Working with the .env file

Adjust important variables in the settings basefile and add them to the .env file. Note that the values in this readme are just some jibberish and not the actual variable values. If you run into error, it is worth double checking the values of the env variables.

##### wsgi.py

**wsgi.py**
Change `os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoku.settings')` to `os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoku.settings.base')`

##### manage.py & .env

**manage.py**
```python
from dotenv import load_dotenv

load_dotenv()
```

Change `os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'djangoku.settings')` to `os.environ.setdefault('DJANGO_SETTINGS_MODULE', os.getenv("DJANGO_SETTINGS_MODULE"))`


**.env**
Add `export DJANGO_SETTINGS_MODULE=djangoku.settings.local_alpha` to your `.env` file.


##### base.py & .env

**djangoku/settings/base.py:**
```python
from dotenv import load_dotenv

load_dotenv()
```

Change `SECRET_KEY = 'AAABBBBCCCC'` to `SECRET_KEY = os.getenv("DJANGO_SECRET_KEY")`

Change:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

to this:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv("DB_NAME"),
        'USER': os.getenv("DB_USER"),
        'PASSWORD': os.getenv("DB_PASSWORD"),
        'HOST': os.getenv("DB_HOST"),
        'PORT': os.getenv("DB_PORT"),
    }
}
```

**.env**
Add:
```
export DJANGO_SECRET_KEY='NEWLY%G3GENERATEDsecret_KEY!'

export DB_NAME=djangoku_local
export DB_USER=philipp
export DB_PASSWORD=
export DB_HOST=localhost
export DB_PORT=5432
```

Test if the configuration works by running the server: `python manage.py runserver`. If the environment variables were loaded correctly, you should see this output:

```
[...]
Django version 2.2.3, using settings 'djangoku.settings.local_alpha'
[...]
```

## Base website workflow

To see if the base mechanics of our website work, add basic models, views, urls, tests and templates:

* Create a template folder: `mkdir -p templates/pages`
* Create a base file: `touch  templates/base.html`
* Create a home file: `touch templates/pages/pages_home.html`
* Create a static folder: `mkdir -p static/css`
* Create a css file: `touch static/css/main.css`

* Add `'templates',` to `TEMPLATES[0]['DIRS']` in `djangoku/settings/base.html`.
* Add `'static/',` to `STATICFILES_DIRS` in `djangoku/settings/base.html` (Note: you may need to create this variable)
* Add `STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')` to `djangoku/settings/base.html`

* Create a forms file: `touch pages/forms.py`
* Create a urls file: `touch pages/urls.py`

* Create a tests folder with test files: `mkdir pages/tests && touch pages/tests/__init__.py pages/tests/test_forms.py pages/tests/test_models.py pages/tests/test_views.py`
* Remove existing tests file: `rm -f pages/tests.py`

Add basic markup and content to the files:

**templates/base.html**
```
{% load static %}

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>Djangoku</title>

    <link rel="stylesheet" href="{% static 'css/main.css' %}" type="text/css" />

</head>

<body>
    <h1>It works!</h1>
    {% block content %}
        <p>(no content loaded)</p>
    {% endblock %}
</body>
</html>
```

**templates/pages/pages_home.html**
```
{% extends 'base.html' %}

{% block content %}
    <p>Content works!</p>
    <form method="POST">
        {% csrf_token %}
        {{form.as_p}}
        <input type="submit" value="Submit form">
    </form>
{% endblock %}
```

**static/css/main.css**
```
:root {
  --color-main-bg: #CDDC39;
}

body {
    background-color: var(--color-main-bg);
}
```

Add basic model:

**pages/models.py**
```
from django.db import models

class Snippet(models.Model):
    name = models.CharField(max_length=100)
    content = models.TextField()

    def __str__():
        return self.name
```

* Make migrations: `python manage.py makemigrations`
* Run migrate: `python manage.py migrate`

Add basic form:

**pages/forms.py**
```
from django.forms import ModelForm
from .models import Snippet

class SnippetForm(ModelForm):
    class Meta:
        model = Snippet
        fields = '__all__'

```

Add basic view:

**pages/views.py**
```
from django.shortcuts import render
from .forms import SnippetForm

def home(request):
    if request.method == 'POST':
        form = SnippetForm(request.POST)
        if form.is_valid():
            form.save()

    content = {
        'form': SnippetForm()
    }
    return render(request, 'pages/pages_home.html', content)
```

Add basic urls:

**djangoku/urls.py**
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('pages.urls'))
]
```

**pages/urls.py**
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.home, name='home'),
]
```

Add basic tests:

**pages/tests/test_models.py**
```
from django.test import TestCase

from pages.models import Snippet

class SnippetsTest(TestCase):

def test_snippet_creation(self):
    Snippet.objects.create(name="Testsnippet", content="Test content of snippet")
    Snippet.objects.create(name="Testsnippet2", content="Test content of snippet")
    self.assertEqual(Snippet.objects.count(), 2)
```

**pages/tests/test_forms.py**
```
from django.test import TestCase

from pages.forms import SnippetForm


class SnippetFormTest(TestCase):

    def test_form_renders_snippet_text_input(self):
        form = SnippetForm()
        self.assertTrue('id="id_name"' in form.as_p())
        self.assertTrue('id="id_content"' in form.as_p())

    def test_form_validity(self):
       form_data = {'name': 'something', 'content': 'something else'}
       form = SnippetForm(data=form_data)
       self.assertTrue(form.is_valid())
```

**pages/tests/test_views.py**
```
from django.test import TestCase, Client
from django.urls import reverse

from pages.models import Snippet

class SnippetViewTest(TestCase):
    def setUp(self):
        self.client = Client()

    def test_home_page(self):
        response = self.client.get(reverse('home'))
        self.assertEquals(response.status_code, 200)
        self.assertTemplateUsed(response, "pages/pages_home.html")

    def test_post_to_home_page(self):
        form_data = {'name': 'something', 'content': 'something else'}
        response = self.client.post(reverse('home'), form_data)
        self.assertEquals(response.status_code, 200)
        self.assertEquals(Snippet.objects.count(), 1)
```
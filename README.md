# Django-Polls_App
Basic Polls App built in Python-Django that lets user take votes.

* U: admin
* E: admin@example.com
* P: django26


## Part 1: Getting Started

### Creating a Project
1. CD to your directory then run the ff code.
```
$ django-admin startproject mysite
```
2. Run server to test.
```
$ python manage.py runserver
```
3. You will see the ff directory structure.
```
mysite/
  manage.py
  mysite/
    __init__.py
    settings.py
    urls.py
    asgi.py
    wsgi.py
```

### Creating the Polls App
1. Create your app, make sure it's in the same directory as **manage.py** and run this command.
```
$ python manage.py startapp polls
```
2. You will see the ff directory structure.
```
polls/
  __init__.py
  admin.py
  apps.py
  migrations/
    __init__.py
  models.py
  tests.py
  views.py
```

### Write the First View
1. Open the file **polls/views.py** and put the ff code.
```
from django.http import HttpResponse


def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
2. Create a URLconf in the polls directory, create a file **urls.py** under polls.
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
```
And include the ff code.
```
from django.urls import path

from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```
3. Point the root URLconf at the polls.urls module.
Open **mysite/urls.py** and add an import for **django.urls.include**.
```
from django.contrib import admin
from django.urls import include, path
```
4. Insert an **include()** in the **urlpatterns** list.
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
5. Run server to test.
```
$ python manage.py runserver
```


## Part 2: Database

### Setting Up the Database
1. Open **mysite/settings.py** and set **TIME_ZONE** to your local time zone.
```
TIME_ZONE = 'Asia/Manila'
```
2. Create the tables in the database by running the ff code.
```
$ python manage.py migrate
```

### Creating Models(Database Layout)
1. Open **polls/models.py** and add the ff code.
```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

### Activating Models
1. Add the **polls app** in our project.
Open **mysite/settings.py** and add the dotted path to the **INSTALLED_APPS** setting.
```
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
2. Run the ff code to tell Django that we've made changes to the models.
```
$ python manage.py makemigrations polls
```
This will create a migration file **polls/migrations/0001_initial.py**.
3. Check what SQL that the migration would run.
Run the ff code.
```
$ python manage.py sqlmigrate polls 0001
```
4. Run the ff code to check for errors.
```
$ python manage.py check
```
5. Run **migrate** again to create the model tables in the database.
```
$ python manage.py migrate
```

### Manually Add Data using the Database API
1. Run the ff code.
```
$ python manage.py shell
```
2. Add data for **question** and **published date**.
```
>>> from polls.models import Choice, Question  # Import the model classes we just wrote.

# No questions are in the system yet.
>>> Question.objects.all()
<QuerySet []>

# Create a new Question.
# Support for time zones is enabled in the default settings file, so
# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
# instead of datetime.datetime.now() and it will do the right thing.
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Save the object into the database. You have to call save() explicitly.
>>> q.save()

# Now it has an ID.
>>> q.id
1

# Access model field values via Python attributes.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

# Change values by changing the attributes, then calling save().
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() displays all the questions in the database.
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```


## Part 6: Adding Style to your Django

### Customize your app's look and feel
1. Create a new folder **static** and file **style.css** under polls.  
```
polls/static/polls/style.css
```
2. Add the ff styling code in style.css (Here we are making all the links in the index.html green.)
```
li a {
  color: green;
}
```
3. Add the ff code on top of polls/templates/polls/index.html.
```
{% load static %}

<link rel="stylesheet"  type="text/css" href="{% static 'polls/style.css' %}">
```
4. Run server to test.
```
$ python manage.py runserver
```

### Adding a Background Image
1. Create a new folder **images** and put your background image file here.
```
polls/static/polls/images/background.gif
```
2. Add the ff code in your style.css.
```
body {
  background: white url("images/background.gif.jpeg") no-repeat;
}
```
3. Run server to test.
```
$ python manage.py runserver
```

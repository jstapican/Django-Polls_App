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
6. Go to http://localhost:8000/polls/ in your browser.


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
3. Open **polls/models.py**
Add a **__str__()** method to both **Question** and **Choice** to make the representation of object understandable instead of **Question object (1)**.
```
from django.db import models

class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```
4. Add a custom method for the date in **polls/models.py**.
Note the addition of **import datetime** and **from django.utils import timezone**.
```
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```
5. Run a new shell by running **python manage.py shell** and run the ff code.
```
>>> from polls.models import Choice, Question

# Make sure our __str__() addition worked.
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>

# Django provides a rich database lookup API that's entirely driven by
# keyword arguments.
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>
>>> Question.objects.filter(question_text__startswith='What')
<QuerySet [<Question: What's up?>]>

# Get the question that was published this year.
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>

# Request an ID that doesn't exist, this will raise an exception.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.

# Lookup by a primary key is the most common case, so Django provides a
# shortcut for primary-key exact lookups.
# The following is identical to Question.objects.get(id=1).
>>> Question.objects.get(pk=1)
<Question: What's up?>

# Make sure our custom method worked.
>>> q = Question.objects.get(pk=1)
>>> q.was_published_recently()
True

# Give the Question a couple of Choices. The create call constructs a new
# Choice object, does the INSERT statement, adds the choice to the set
# of available choices and returns the new Choice object. Django creates
# a set to hold the "other side" of a ForeignKey relation
# (e.g. a question's choice) which can be accessed via the API.
>>> q = Question.objects.get(pk=1)

# Display any choices from the related object set -- none so far.
>>> q.choice_set.all()
<QuerySet []>

# Create three choices.
>>> q.choice_set.create(choice_text='Not much', votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text='The sky', votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

# Choice objects have API access to their related Question objects.
>>> c.question
<Question: What's up?>

# And vice versa: Question objects get access to Choice objects.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3

# The API automatically follows relationships as far as you need.
# Use double underscores to separate relationships.
# This works as many levels deep as you want; there's no limit.
# Find all Choices for any question whose pub_date is in this year
# (reusing the 'current_year' variable we created above).
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

# Let's delete one of the choices. Use delete() for that.
>>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
>>> c.delete()
```

### Setting Up the Admin Site
1. Run the ff code to create an admin user.
```
$ python manage.py createsuperuser
```
2. Enter the desired username, email address and password.
```
Username: admin
Email address: admin@example.com
Password: **********
Password (again): *********
Superuser created successfully.
```
3. Run the ff code to start the development server.
```
$ python manage.py runserver
```
4. Go to http://127.0.0.1:8000/admin/ in your browser.
5. Login to the superuser account using the above credentials.
6. Make the Poll App modifiable in the admin.
Open the **polls/admin.py** and add the ff code.
This will tell the admin that **Question** objects have an admin interface.
```
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```
7. Refresh the admin index page in your browser to check if all are ok.
8. Click **Questions** then **What's up?** in your admin.
Change the **Date publlished** by clicking the **Today** and **Now** shortcuts.
Click **Save and continue editing"** then click **History** to check all the changes we've made.


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

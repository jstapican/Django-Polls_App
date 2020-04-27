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


## Part 3: Adding More Views, 404 Error, Removing Hardcoded URLs and Namespacing URL Names

### Adding More Views
1. Open **polls/views.py** and add the ff code.
```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
2. Open **polls/urls.py** and add the following **path()** calls.
```
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
3. Open **polls/view** and add the ff code.
This will update the **index()** view which will display the latest 5 poll questions in the system, separated by commas, according to publication date.
```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged
```
4. Separate the design from Python by creating a template.
Create **polls/templates/polls** directory.
5. Create a new file **polls/templates/polls/index.html** and put the ff code.
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```
6. Update our **index** view in **polls/views.py** to use the template.
```
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
7. Use **render()** as a shortcut for loading a template, filling a context and returning an **HttpResponse** object with the result of the rendered template.
Update the **index** view again with the ff code.
```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

### Raising a 404 Error
1. Add the ff code in **polls/views.py**.
```
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```
2. Add the ff code in **polls/detail.html** to test.
```
{{ question }}
```
3. It's very common to use **get()** and raise **Http404** if the object doesn't exist.
We can use **get_object_or_404** for this in the updated **polls/views.py**.
```
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```

### Use the Template system
1. Open **polls/templates/polls/detail.html** and add the ff code.
```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```

### Removing Hardcoded URLs in the TEMPLATES
1. Open **polls/index.html** and change the hardcoded code from
```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```
to
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
2. Open **polls/urls.py**. Make sure the ff code is there.
```
...
# the 'name' value as called by the {% url %} template tag
path('<int:question_id>/', views.detail, name='detail'),
...
```

### Namespacing URL Names
1. Add namespaces to your URLconf to help Django know which app view to create for a url when using the **{% url %}**.
Open **polls/urls.py** and add **app_name**.
```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.index, name='index'),
    path('<int:question_id>/', views.detail, name='detail'),
    path('<int:question_id>/results/', views.results, name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
2. Change **polls/index.html** from
```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```
to
```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```


## Part 4: Form Processing and Code Cutting

### Write a Minimal form
1. Open **polls/templates/polls/detail.html** and add an HTML **<form>** element.
```
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```
2. Add the ff code in **polls/views.py**.
```
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```
3. Add the ff code for results view.
```
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```
4. Create **polls/results.html** template.
```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

### Use Generic views
1. Amend URLconf. Open **polls/urls.py** and update it with the ff code.
```
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
2. Amend views. Open **polls/views.py** and update it with the ff code.
```
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```


## Part 5: Automated Tests

### Writing our First Test (Tests for Internal Behavior)
1. There's a bug in the polls application where it returns **True** if the **Question** was published in the future.
Run shell command.
```
$ python manage.py shell
```
2. Run the ff code to test the bug. Future questions are returning as **True** which is wrong.
```
>>> import datetime
>>> from django.utils import timezone
>>> from polls.models import Question
>>> # create a Question instance with pub_date 30 days in the future
>>> future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))
>>> # was it published recently?
>>> future_question.was_published_recently()
True
```
3. Open **polls/tests.py** and add the ff code that will test for the bug.
```
import datetime

from django.test import TestCase
from django.utils import timezone

from .models import Question


class QuestionModelTests(TestCase):

    def test_was_published_recently_with_future_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is in the future.
        """
        time = timezone.now() + datetime.timedelta(days=30)
        future_question = Question(pub_date=time)
        self.assertIs(future_question.was_published_recently(), False)
```
4.  Run tests. The test informs us which test failed and even the line on which the failure occurred.
```
$ python manage.py test polls
```
5. After we identify the error, let's fix it.
Add the ff code in **polls/models/py**, so that it will only return **True** if the date is also in the past.
```
def was_published_recently(self):
    now = timezone.now()
    return now - datetime.timedelta(days=1) <= self.pub_date <= now
```
6. Run the test again to verify if the bug has been fixed.
```
$ python manage.py test polls
```
7. Additional tests. Now we have three tests that confirm that **Question.was_published_recently()** returns sensible values for past, recent, and future questions.
```
def test_was_published_recently_with_old_question(self):
    """
    was_published_recently() returns False for questions whose pub_date
    is older than 1 day.
    """
    time = timezone.now() - datetime.timedelta(days=1, seconds=1)
    old_question = Question(pub_date=time)
    self.assertIs(old_question.was_published_recently(), False)

def test_was_published_recently_with_recent_question(self):
    """
    was_published_recently() returns True for questions whose pub_date
    is within the last day.
    """
    time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
    recent_question = Question(pub_date=time)
    self.assertIs(recent_question.was_published_recently(), True)
```

### Testing for User Experience Behavior
1. Let's setup a test environment in the shell.
Run first the ff code.
```
$ python manage.py shell
```
Then
```
>>> from django.test.utils import setup_test_environment
>>> setup_test_environment()
```
```
>>> from django.test import Client
>>> # create an instance of the client for our use
>>> client = Client()
```
```
>>> # get a response from '/'
>>> response = client.get('/')
Not Found: /
>>> # we should expect a 404 from that address; if you instead see an
>>> # "Invalid HTTP_HOST header" error and a 400 response, you probably
>>> # omitted the setup_test_environment() call described earlier.
>>> response.status_code
404
>>> # on the other hand we should expect to find something at '/polls/'
>>> # we'll use 'reverse()' rather than a hardcoded URL
>>> from django.urls import reverse
>>> response = client.get(reverse('polls:index'))
>>> response.status_code
200
>>> response.content
b'\n    <ul>\n    \n        <li><a href="/polls/1/">What&#x27;s up?</a></li>\n    \n    </ul>\n\n'
>>> response.context['latest_question_list']
<QuerySet [<Question: What's up?>]>
```

### Improving our View
1. The list of polls shows polls that aren’t published yet (i.e. those that have a pub_date in the future). Let’s fix that.
Open **polls/views.py** and add the ff code.
```
from django.utils import timezone
```
```
def get_queryset(self):
    """
    Return the last five published questions (not including those set to be
    published in the future).
    """
    return Question.objects.filter(
        pub_date__lte=timezone.now()
    ).order_by('-pub_date')[:5]
```

### Testing our New Views
1. Open **polls/tests.py** and add the ff code.
```
from django.urls import reverse
```
2. Create a shortcut function to create questions as well as a new test class.
```
def create_question(question_text, days):
    """
    Create a question with the given `question_text` and published the
    given number of `days` offset to now (negative for questions published
    in the past, positive for questions that have yet to be published).
    """
    time = timezone.now() + datetime.timedelta(days=days)
    return Question.objects.create(question_text=question_text, pub_date=time)


class QuestionIndexViewTests(TestCase):
    def test_no_questions(self):
        """
        If no questions exist, an appropriate message is displayed.
        """
        response = self.client.get(reverse('polls:index'))
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_past_question(self):
        """
        Questions with a pub_date in the past are displayed on the
        index page.
        """
        create_question(question_text="Past question.", days=-30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_future_question(self):
        """
        Questions with a pub_date in the future aren't displayed on
        the index page.
        """
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertContains(response, "No polls are available.")
        self.assertQuerysetEqual(response.context['latest_question_list'], [])

    def test_future_question_and_past_question(self):
        """
        Even if both past and future questions exist, only past questions
        are displayed.
        """
        create_question(question_text="Past question.", days=-30)
        create_question(question_text="Future question.", days=30)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question.>']
        )

    def test_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        create_question(question_text="Past question 1.", days=-30)
        create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse('polls:index'))
        self.assertQuerysetEqual(
            response.context['latest_question_list'],
            ['<Question: Past question 2.>', '<Question: Past question 1.>']
        )
```

### Testing for the DetailView.
1. Even though future questions don’t appear in the index, users can still reach them if they know or guess the right URL.
So we need to add a similar constraint to **DetailView**. Open **polls/views.py** and add the ff code.
```
class DetailView(generic.DetailView):
    ...
    def get_queryset(self):
        """
        Excludes any questions that aren't published yet.
        """
        return Question.objects.filter(pub_date__lte=timezone.now())
```
2. Let's add a test to check that a **Question** whose **pub_date** is in the past can be displayed, and that one with a pub_date in the future is not.
```
class QuestionDetailViewTests(TestCase):
    def test_future_question(self):
        """
        The detail view of a question with a pub_date in the future
        returns a 404 not found.
        """
        future_question = create_question(question_text='Future question.', days=5)
        url = reverse('polls:detail', args=(future_question.id,))
        response = self.client.get(url)
        self.assertEqual(response.status_code, 404)

    def test_past_question(self):
        """
        The detail view of a question with a pub_date in the past
        displays the question's text.
        """
        past_question = create_question(question_text='Past Question.', days=-5)
        url = reverse('polls:detail', args=(past_question.id,))
        response = self.client.get(url)
        self.assertContains(response, past_question.question_text)
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

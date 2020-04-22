# Django-Polls_App
Basic Polls App built in Python-Django that lets user take votes.

* U: admin
* E: admin@example.com
* P: django26

## Part 6: Adding Style to your Django

### Customize your app's look and feel
1. Create a new folder and file (style.css) under polls.  
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
python manage.py runserver
```

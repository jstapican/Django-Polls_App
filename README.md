# Django-Polls_App
Basic Polls App built in Python-Django that lets user take votes.

* U: admin
* E: admin@example.com
* P: django26

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
python manage.py runserver
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
python manage.py runserver
```

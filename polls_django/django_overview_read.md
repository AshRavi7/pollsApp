# Django-python framework 

Django was designed to make common Web-development tasks fast and easy

You can use Django without a database, it comes with an object-relational mapper in which you describe your database layout in Python code. The data-model syntax offers many rich ways of representing your models – so far, it’s been solving many years’ worth of database-schema problems.  
For example-

```from django.db import models

class Student(models.Model):
    full_name = models.CharField(max_length=70)

    def __str__(self):
        return self.full_name

class Blog(models.Model):
    pub_date = models.DateField()
    tagline = models.CharField(max_length=200)
    content = models.TextField()
    blog_master = models.ForeignKey(Student, on_delete=models.CASCADE)

    def __str__(self):
        return self.tagline  
 ```
 
 Run the Django command-line utilities to create the database tables automatically: 
 py manage.py makemigrations
 py manage.py migrate
 
The makemigrations command looks at all your available models and creates migrations for whichever tables don’t already exist. migrate runs the migrations and creates tables in your database, as well as optionally providing much richer schema control.

We’ve got a free, and rich, Python API to access your data.

```# Import the models we created from our app
>>> from app_name.models import Student,Blog

# No students are in the system yet.
>>> Student.objects.all()
<QuerySet []>

# Create a new student.
>>> s=Student(full_name='Ashwin Ravi')

# Save the object into the database. You have to call save() explicitly.
>>> s.save()

# Now it has an ID.
>>> s.id
1

# Now the new student is in the database.
>>> Student.objects.all()
<QuerySet [<Student: Ashwin Ravi>]>

# Fields are represented as attributes on the Python object.
>>> s.full_name
'Ashwin Ravi'

# Django provides a rich database lookup API.
>>> Student.objects.get(id=1)
<Reporter: John Smith>
>>> Student.objects.get(full_name__startswith='Ashwin')
<Student:Ashwin Ravi>
>>> Student.objects.get(full_name__contains='avi')
<Student:Ashwin Ravi>
>>> Student.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Student matching query does not exist.

# Create a Blog.
>>> from datetime import date
>>> a = Blog(pub_date=date.today(), tagline='Django not bad',
...     content='xperiment!', blog_user=s)
>>> a.save()

# Now the Blog is in the database.
>>> Blog.objects.all()
<QuerySet [<Article: Django not bad>]>

# Blog objects get API access to related Student objects.
>>> s = a.blog_user
>>> s.full_name
'Ashwin Ravi'

# And vice versa: Student objects get API access to Blog objects.
>>> s.blog_set.all()
<QuerySet [<Article: Django not bad>]>

# The API follows relationships as far as you need, performing efficient
# JOINs for you behind the scenes.
# This finds all articles by a reporter whose name starts with "John".
>>> Blog.objects.filter(blog_user__full_name__startswith='Ashwin')
<QuerySet [<Article: Django not bad>]>

# Change an object by altering its attributes and calling save().
>>> s.full_name = 'god'
>>> s.save()

# Delete an object with delete().
>>> s.delete()
```

Once your models are defined, Django can automatically create a professional, production ready administrative interface – a website that lets authenticated users add, change and delete objects. The only step required is to register your model in the admin site

```-->admin.py 
from django.contrib import admin

from . import models

admin.site.register(models.Student)
admin.site.register(models.Blog)
```
#### The philosophy here is that your site is edited by a staff, or a client, or maybe just you – and you don’t want to have to deal with creating backend interfaces only to manage content.

#### One typical workflow in creating Django apps is to create models and get the admin sites up and running as fast as possible, so your staff (or clients) can start populating data. Then, develop the way data is presented to the public.

## Design your URLs

A clean, elegant URL scheme is an important detail in a high-quality Web application.To design URLs for an app, you create a Python module called a URLconf. A table of contents for your app, it contains a mapping between URL patterns and Python callback functions. 

```from django.urls import path

from . import views

urlpatterns = [
    path('blog/<int:year>/', views.year_archive),
]
```
The code above maps URL paths to Python callback functions (“views”). The path strings use parameter tags to “capture” values from the URLs. When a user requests a page, Django runs through each path, in order, and stops at the first one that matches the requested URL. (If none of them matches, Django calls a special-case 404 view.) This is blazingly fast, because the paths are compiled into regular expressions at load time.

Once one of the URL patterns matches, Django calls the given view, which is a Python function. Each view gets passed a request object – which contains request metadata – and the values captured in the pattern.
For example, if a user requested the URL “/blog/2020”, Django would call the function app_name.views.blog_detail(request, year=2020).

## Write your views¶

Each view is responsible for doing one of two things: Returning an HttpResponse object containing the content for the requested page, or raising an exception such as Http404. The rest is up to you.

Generally, a view retrieves data according to the parameters, loads a template and renders the template with the retrieved data. Here’s an example view for year_archive from above:

```from django.shortcuts import render

from .models import Blog

def year_archive(request, year):
    a_list = Blog.objects.filter(pub_date__year=year)
    context = {'year': year, 'article_list': a_list}
    return render(request, 'app_name/year_archive.html', context)
 ```
    
    
```{% extends "base.html" %}

{% block title %}Blogs for {{ year }}{% endblock %}

{% block content %}
<h1>Blogs for {{ year }}</h1>

{% for article in article_list %}
    <p>{{ blog.tagline }}</p>
    <p>By {{ blog.blog_user.full_name }}</p>
    <p>Published {{ blog.pub_date|date:"F j, Y" }}</p>
{% endfor %}
{% endblock %}
```

Variables are surrounded by double-curly braces. {{ blog.tagline }} means “Output the value of the blog’s tagline attribute.” But dots aren’t used only for attribute lookup. They also can do dictionary-key lookup, index lookup and function calls.

Note {{ blog.pub_date|date:"F j, Y" }} uses a Unix-style “pipe” (the “|” character). This is called a template filter, and it’s a way to filter the value of a variable.

Django uses the concept of “template inheritance”. That’s what the {% extends "base.html" %} does. It means “First load the template called ‘base’, which has defined a bunch of blocks, and fill the blocks with the following blocks.” In short, that lets you dramatically cut down on redundancy in templates: each template has to define only what’s unique to that template.

```-> django_project/templates/base.html
{% load static %}
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <img src="{% static 'images/sitelogo.png' %}" alt="Logo">
    {% block content %}{% endblock %}
</body>
</html>
```

Django defines the look-and-feel of the site (with the site’s logo), and provides “holes” for child templates to fill. This means that a site redesign can be done by changing a single file – the base template.

It also lets you create multiple versions of a site, with different base templates, while reusing child templates. Django’s creators have used this technique to create strikingly different mobile versions of sites by only creating a new base template.

Package structure --
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        asgi.py
        wsgi.py
    polls/
        __init__.py
        admin.py
        apps.py
        migrations/
            __init__.py
            0001_initial.py
        models.py
        static/
            polls/
                images/
                    background.gif
                style.css
        templates/
            polls/
                detail.html
                index.html
                results.html
        tests.py
        urls.py
        views.py
    templates/
        admin/
            base_site.html
```
Installation Process -
1.Install Python
2.Install Djangp(pip install django if you are using pip package manager) 
3.Verifying 
```
>>> import django
>>> print(django.get_version())
```
Begin journey ! All the best ! 

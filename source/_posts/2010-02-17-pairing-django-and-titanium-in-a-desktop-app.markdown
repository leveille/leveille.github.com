---
layout: post
title: "Pairing Django and Titanium in a Desktop App"
date: 2010-02-17 21:07
comments: false
categories: Django
---

Over the past four months I have been doing a lot of work in Python and Django. Naturally, while recently playing around with [Appcelerator Titanium Desktop](http://www.appcelerator.com/products/titanium-desktop-application-development/) (which seems to have good Python support), I wanted to see how Django and Titanium would pair.

##Assumptions

1. [Titanium installed](http://www.appcelerator.com/products/download/)
2. [Django installed](http://docs.djangoproject.com/en/dev/topics/install/)

##What Worked

Over the course of a few hours, I was able to accomplish the following:

* Django Template integration
* Django ORM interaction
* Titanium Database communication with the Django DB (SQLite) defined in settings.py

##Links

* [Source Files](/assets/leveille-django-titanium-7da2c38.tar.gz)

##Directory Layout

{% img https://www.evernote.com/shard/s33/sh/06d56a74-d35e-419e-840f-13cc9ab3dc22/9e3f959c495e5f348d463c6882dd7f75/res/8d23c11d-590c-404f-a1e7-d5b3dd480110.jpg?resizeSmall&width=832 %}

##Creating the Project

When a Titanium Desktop application (with Python support) is running, the Resources directory for the application is placed in the system Python path. Therefore, my first step was to place the main django directory in Resources. From there, I navigated to Resources and executed the startproject command:

```
$ django-admin.py startproject test_project
```

Next, I updated settings.py with information about my test database:

``` python
import os
DATABASES ={
    'default':{
        'ENGINE':'django.db.backends.sqlite3',
        'NAME': os.path.join(os.path.dirname(os.path.dirname(__file__)),'test_project.db'),
    }
}
```

I made sure to include the full path to my test_project.db database, knowing that I would eventually need to refer to this path for Titanium Database support. Finally, I executed my syncdb command from within my test_project directory:

```
$ ./manage.py syncdb
```

When prompted, I added a superuser account for myself.

##Django Template Integration

Template integration was pretty smooth. What follows is a scaled down version of my index.html resource, and its associated files:

Resources >> **index.html**

``` javascript
//Yes,this is all I have inmy main index.html resource
<script type="text/python">             
    from test_project.contrib.template import index
    document.write(index())
</script>
```

Resources >> test_project >> contrib >> **template.py**

``` python
def index():
    kwargs ={'foo':'bar'}
    return _render_template('index',**kwargs)

def _render_template(template_name,**kwargs):
    from django.templateimport loader,Context
    t = loader.get_template('%s.html'% template_name)
    c =Context(kwargs)
    return t.render(c)
```

Resources >> test_project >> templates >> **base.html**, **index.html**

{% gist 4440297 %}

{% gist 4440303 %}

##Database Communication

Database communication also went fairly smooth (though I did get tripped up trying to find a method to use an existing SQLite db for Titanium ... ultimately the following worked for me: 

```
Titanium.Database.openFile('/path/to/db'))
```

What follows are the code snippets which represent successful communication with the DB via Django, and via the Titanium Database API:

Resources >> test_project >> contrib >> **db.py**, **user.py**

{% gist 4440318 %}

{% gist 4440319 %}

Resources >> test_project >> templates >> **index.html** (full version)

{% gist 4440328 %}

##The Result

{% img https://www.evernote.com/shard/s33/sh/06d56a74-d35e-419e-840f-13cc9ab3dc22/9e3f959c495e5f348d463c6882dd7f75/res/bba35d6b-fa73-45f5-873b-e278c340cda5.jpg?resizeSmall&width=832 %}

If you have experience playing around with these two playfellows, feel free to share them here.

---
layout: post
title: What is a Django app?
description: "Apps are a fundamental concept in Django. But what exactly are they and when should we use them?"
image: installed-apps.jpg
featured: false
weight: 2
tags: [django]
---
If you've ever built anything using Django, you'll be used to including apps in your `INSTALLED_APPS` setting:

{% highlight python %}
# settings.py

INSTALLED_APPS = [
    'django.contrib.auth',
    'myapp',
]
{% endhighlight %}

This is something we take for granted, but it's not always obvious why we need to do this. For something we use so routinely, it's a concept we're surprisingly vague about.

So, what are Django apps for? Why do we need to install them? Why can't they just be plain old Python packages? Before you read on, think for a moment and try to answer these questions for yourself.

# What apps are for: a general rule

Here's a general rule for the purpose of a Django app:
 
<div class='rule'>
  Apps are for introspection.
</div>

When you declare a particular Python package as an app (by including it in `INSTALLED_APPS`), you allow your project to 'look into itself' (i.e. introspect) and discover and enable functionality within that package.

By way of contrast, look at an aspect of Django that *doesn't* use the apps system: middleware. Middleware classes can sit anywhere in your project, but they need to be explicitly configured:

{% highlight python %}
MIDDLEWARE = [
    'myapp.middleware.MyMiddleware',
]
{% endhighlight %}

In contrast, features which use the apps system (e.g. models) don't need any specific configuration to let the framework know about them other than app installation.

## Some examples

1. *Models:* The most obvious one; Django looks for any `models` module within your package and handles the necessary database migrations.
2. *Admin:* You register models in the Django Admin Site within an `admin` module.   
3. *Template tags:* You may include a `templatetags` package to allow loading of custom template tags.
4. *Template directories:* Assuming you have configured your `TEMPLATES` setting with `APP_DIRS` as `True`, Django will know about any files within a `templates` subdirectory in your app.
5. *Celery tasks:* If you're using [Celery](http://docs.celeryproject.org/){:target="_blank"}, you can include your tasks in a `tasks.py` and Celery will automatically pick them up.
 
# Autodiscovery

Notice that all of these examples follow a similar pattern: for each piece of functionality, Django looks in every installed app for a particular module/directory name; if it's there, it will use it. This is the foundation of Django's pluggable architecture; providing a package is in your Python path, adding it to `INSTALLED_APPS` allows it to hook in to other parts of the system.

This process is called **autodiscovery**, and there is a built in function for it: `django.utils.module_loading.autodiscover_modules`. To use, simply call it with the name of the module you want, and Django will import any modules that have that name (providing they are in `INSTALLED_APPS`, of course). For example, to import every module named `foo.py`, just call:

{% highlight python %}
from django.utils.module_loading import autodiscover_modules

autodiscover_modules('foo')
{% endhighlight %}

{% include tips/open.html %}
    <p>
      A good place to call <code>autodiscover_modules</code> is in the
      <code>ready()</code> method of your application configuration class.
    </p>
    {% highlight python %}
# myapp/__init__.py

from django.apps import AppConfig as BaseAppConfig
from django.utils.module_loading import autodiscover_modules


class AppConfig(BaseAppConfig):
    name = 'myapp'

    def ready(self):
        autodiscover_modules('products')


default_app_config = 'myapp.AppConfig'
    {% endhighlight %}
{% include tips/close.html %}


# Understanding INSTALLED_APPS can improve your Django

Why is knowing this useful? If you understand the fundamental concept of what an app is, you should be able to structure your projects better. Apps should be pluggable, discoverable modules, so design them accordingly.

That means if you have a large amount of interdependent functionality, there really is no need to split it up into multiple apps. You're probably better off splitting up the code into a manageable file structure. For example, there's nothing special about `models.py` other than it gets automatically imported during certain model-related tasks. To make things more manageable, you could turn `models.py` into a subpackage within your app, distribute the models within different files inside it, and import those models within the `models/__init__.py`.

Conversely, if parts of your project could be designed as plugins, consider making them apps (even if they don't declare any models). Let's say you have a site that defines a range of different products by inheriting from a base `Product` class. Rather than manually importing them, or configuring them in your settings, you could have your core products app autodiscover modules named `products`. This would allow you to have separate, per-product apps that declare their products in `products.py` files. (For more ideas about this, you may like to watch my talk on [Encapsulated Django]({% link _talks/2015-11-16-encapsulated-django.md %}).)

To sum up, the concept of a Django app is a powerful one that goes beyond mere models. It's the magic that allows the framework to find out about what's inside your project. Knowing this can help you understand how certain things get imported, and give you confidence to be inventive in the way you architect your code. 

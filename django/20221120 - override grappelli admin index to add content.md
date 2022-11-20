---
title: Overriding Django Grappelli to add additional content to admin index
slug: overriding-django-grappelli-to-add-additional-content-to-admin-index
tags: django
domain: til.jacklinke.com
---

## The Goal

I wanted to override django-grappelli's admin index page in our project with some additional context. Specifically, [Watervize](https://www.watervize.com/) has multiple tenants, and each has a different url (e.g.: *tenant.watervize<i></i>.com* or *order.tenant<i></i>.com*). I wanted a list of each tenant and a link to its associated site and admin urls at the top of my admin index to keep these visible to our internal admins and to facilitate navigation from one tenant site to another.

I am using django-grappelli, but the same general technique will work for django's default admin or other admin packages, understanding that you will have to adjust the html to match the package you're working with.

Here is the final look of the top of my admin index page:

![Final Product](https://raw.githubusercontent.com/jacklinke/til/main/django/20221120-override-grappelli-admin-index-final-product.png)


## Class Structure

Watervize tenants are referred to as "Providers", and each has a unique url from which they serve customers.

The Provider model includes a `website` field which allows entries in the form of `subdomain.domain.com`. *Note: these entries are only added or updated by Watervize admins.*

```python
class PartialURLField(CharField):
    default_validators = [
        RegexValidator(
            "^[a-z0-9]+([\-\.]{1}[a-z0-9]+)*\.[a-z]{2,3}$",
            message=_("please enter the url in this format: subdomain.domain.com"),
        )
    ]
    description = _("URL")

    def __init__(self, verbose_name=None, name=None, **kwargs):
        kwargs.setdefault("max_length", 100)
        super().__init__(verbose_name, name, **kwargs)


class Provider(models.Model):
    # ...

    website = PartialURLField(_("Website"), default="", unique=True)

    # ..
```

## Context

In order to display details about tenants on the admin index page, we needed a way to get the information there. There are a few different ways one might approach this:


- Override the [admin views](https://docs.djangoproject.com/en/4.1/ref/contrib/admin/#customizing-the-adminsite-class). This seemed a bit heavy-handed for the simple change we wanted to implement.
- [Monkey patch](https://en.wikipedia.org/wiki/Monkey_patch) the admin index view. This almost *never* the ideal way. Monkey patching is powerful, but dangerous and prone to issues that will pop up later and bite you when updating django.
- Create a [context processor](https://docs.djangoproject.com/en/4.1/ref/templates/api/#using-requestcontext) which allows us to populate requests with some additional context.

We went with that last option. From the docs regarding context processors:

> A context processor has a simple interface: It’s a Python function that takes one argument, an HttpRequest object, and returns a dictionary that gets added to the template context.


At the most basic, we can use the following, which will return a context dictionary with each Provider and its associated site and admin urls. Note that we are formatting the urls differently depending on whether or not django is being run in `DEBUG`. Your setup and requirements may be different.

```python
from django.conf import settings
from watervize.entities.models import Provider


def provider_site_admin_context():
    """Return urls for all Provider home and admin index pages"""
    context = {}

    for provider in Provider.objects.all():
        if settings.DEBUG:
            context[provider.name] = {
                "site": f"http://{provider.website}:8000",
                "admin": f"http://{provider.website}:8000{admin_index}",
            }
        else:
            context[provider.name] = {
                "name": provider.name,
                "site": f"https://{provider.website}",
                "admin": f"https://{provider.website}{admin_index}",
            }

    return {"provider_site_admin_context": context}
```

The above is less than optimal, because context will be injected into **every** request throughout our project, but we only need it on the admin index page.

To fix this issue, we can check whether the current path for a request matches the path for the admin index page by using [`reverse`](https://docs.djangoproject.com/en/4.1/ref/urlresolvers/#reverse) to retrieve the path for the admin index view. We are using reverse here instead of hard-coding `admin/`, because we prefer to set the path for admin in our settings, allowing it to vary from environment to environment. If the path matches, we process and return the Provider information. Otherwise, we just return an empty context dictionary.

*Note: To find the path, view, and namespace & name for all of your urls, I always recommend installing [django-extensions](https://django-extensions.readthedocs.io/en/latest/). Then you can run `python manage.py show_urls` for a listing of **all** valid urls in your project. Django-extensions has a number of other very useful utilities as well.*

```python
from django.conf import settings
from django.urls import reverse
from watervize.entities.models import Provider

def provider_site_admin_context():
    """Return urls for all Provider home and admin index pages"""
    context = {}
    admin_index = reverse("admin:index")

    if request.path == admin_index:  # current path matches admin index page
        for provider in Provider.objects.all():
            if settings.DEBUG:
                context[provider.name] = {
                    "site": f"http://{provider.website}:8000",
                    "admin": f"http://{provider.website}:8000{admin_index}",
                }
            else:
                context[provider.name] = {
                    "name": provider.name,
                    "site": f"https://{provider.website}",
                    "admin": f"https://{provider.website}{admin_index}",
                }
    return {"provider_site_admin_context": context}
```

Now, if the request's path matches the admin index path, the context processor will add the following context dictionary to the page's context:

```python
'provider_site_admin_context': {
    'Some Water District': {
        'site': 'http://swd.mylocalsite.io:8000',
        'admin': 'http://swd.mylocalsite.io:8000/admin/'
    },
    'Another Cool District': {
        'site': 'http://acd.mylocalsite.io:8000',
        'admin': 'http://acd.mylocalsite.io:8000/admin/'
    },
}

```

A context processor can live anywhere in your project. You just have to be sure to add a reference to it in your project's settings. If we put the context processor function in `context_processors.py` within the `appname` app of our project `myproject`, we would add the following entry to our settings:

```python
TEMPLATES = [
    {
        # Other template settings...

        "OPTIONS": {
            # Other template options settings...

            "context_processors": [
                "myproject.appname.context_processors.provider_site_admin_context",
            ],
        },
    }
]
```

## The Template

Django-Grappelli's [admin index page](https://github.com/sehmaschine/django-grappelli/blob/master/grappelli/templates/admin/index.html) extends from django's admin index page. We, in turn, extend our page from Grappelli's. More information about how to structure templates and how to extend them can be found at [the django docs](https://docs.djangoproject.com/en/4.1/howto/overriding-templates/) and this [good description of template structure best practices](https://learndjango.com/tutorials/template-structure) by [Will Vincent](https://wsvincent.com/).

Within our project's templates folder, we create an `admin` directory if it is not already present. Then we add `index.html`. Django will use this template for the admin index, since it is the lowest level (templates within our project first, then 3rd party templates, then Django's default templates).

Here we are using the same html classes and div structure used by django-grappelli in order to keep the visual appearance the same. We then use `{{ block.super }}` to load up the rest of the contents django-grappelli normally loads in the admin index page.



```html
<!-- Override Grappelli's index to include details about the Providers served in this instance of Watervize -->
{% extends "admin/index.html" %}

<!-- LOADING -->
{% load i18n grp_tags log %}

<!-- CONTENT -->
{% block content %}
    <div class="g-d-c">
        <div class="g-d-12 g-d-f">
            <div class="grp-module">
                <h2>Providers</h2>
                {% for provider_name, provider_details in provider_site_admin_context.items %}
                    <div class="grp-row">
                        {{ provider_name }}:
                        <div class="grp-actions">
                            <a href="{{ provider_details.site }}" title="{% trans 'Go to Provider Site' %}">Site</a> |
                            <a href="{{ provider_details.admin }}" title="{% trans 'Go to Provider Admin' %}">Admin</a>
                        </div>
                    </div>
                {% endfor %}
            </div>
        </div>
    </div>

    {{ block.super }}
{% endblock %}

```

And that's it. Now, when we go to the admin index page it will show all of our Providers' names, along with links to their main site and admin urls.

![Final Product Again](https://raw.githubusercontent.com/jacklinke/til/main/django/20221120-override-grappelli-admin-index-final-product.png)

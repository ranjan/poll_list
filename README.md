We’ll assume you have Django installed already. You can tell Django is installed and which version by running the following command:

python -c "import django; print(django.get_version())"

Creating a project

django-admin.py startproject mysite

The development server

python manage.py runserver

Database setup

Edit mysite/settings.py.

By default, INSTALLED_APPS contains the following apps, all of which come with Django:

    django.contrib.auth -- An authentication system.
    django.contrib.contenttypes -- A framework for content types.
    django.contrib.sessions -- A session framework.
    django.contrib.sites -- A framework for managing multiple sites with one Django installation.
    django.contrib.messages -- A messaging framework.
    django.contrib.staticfiles -- A framework for managing static files.

Each of these applications makes use of at least one database table, though, so we need to create the tables in the database before we can use them. To do that, run the following command:

python manage.py syncdb


To create your app, make sure you're in the same directory as manage.py and type this command:

python manage.py startapp polls

Edit the polls/models.py file so it looks like this:

from django.db import models

class Poll(models.Model):
    question = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    poll = models.ForeignKey(Poll)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField()

Activating models¶

Edit the settings.py file again, and change the INSTALLED_APPS setting to include the string 'polls'. So it'll look like this:

INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Uncomment the next line to enable the admin:
    # 'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'polls',
)

python manage.py sql polls

You should see something similar to the following (the CREATE TABLE SQL statements for the polls app):

BEGIN;
CREATE TABLE "polls_poll" (
    "id" serial NOT NULL PRIMARY KEY,
    "question" varchar(200) NOT NULL,
    "pub_date" timestamp with time zone NOT NULL
);
CREATE TABLE "polls_choice" (
    "id" serial NOT NULL PRIMARY KEY,
    "poll_id" integer NOT NULL REFERENCES "polls_poll" ("id") DEFERRABLE INITIALLY DEFERRED,
    "choice_text" varchar(200) NOT NULL,
    "votes" integer NOT NULL
);
COMMIT;

Now, run syncdb again to create those model tables in your database:

python manage.py syncdb

Use Console

python manage.py shell

In the polls/models.py file) and adding a __unicode__() method to both Poll and Choice:

class Poll(models.Model):
    # ...
    def __unicode__(self):
        return self.question

class Choice(models.Model):
    # ...
    def __unicode__(self):
        return self.choice_text


Activate the admin site¶

The Django admin site is not activated by default – it’s an opt-in thing. To activate the admin site for your installation, do these three things:

    Uncomment "django.contrib.admin" in the INSTALLED_APPS setting.

    Run python manage.py syncdb. Since you have added a new application to INSTALLED_APPS, the database tables need to be updated.

    Edit your mysite/urls.py file and uncomment the lines that reference the admin – there are three lines in total to uncomment. This file is a URLconf; we’ll dig into URLconfs in the next tutorial. For now, all you need to know is that it maps URL roots to applications. In the end, you should have a urls.py file that looks like this:

    from django.conf.urls import patterns, include, url

    # Uncomment the next two lines to enable the admin:
    from django.contrib import admin
    admin.autodiscover()

    urlpatterns = patterns('',
        # Examples:
        # url(r'^$', '{{ project_name }}.views.home', name='home'),
        # url(r'^{{ project_name }}/', include('{{ project_name }}.foo.urls')),

        # Uncomment the admin/doc line below to enable admin documentation:
        # url(r'^admin/doc/', include('django.contrib.admindocs.urls')),

        # Uncomment the next line to enable the admin:
        url(r'^admin/', include(admin.site.urls)),
    )

create a file called admin.py in your polls directory, and edit it to look like this:

from django.contrib import admin
from polls.models import Poll

admin.site.register(Poll)

Let's see how this works by re-ordering the fields on the edit form. Replace the admin.site.register(Poll) line with:

class PollAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question']

admin.site.register(Poll, PollAdmin)

you might want to split the form up into fieldsets:

class PollAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question']}),
        ('Date information', {'fields': ['pub_date']}),
    ]

admin.site.register(Poll, PollAdmin

You can assign arbitrary HTML classes to each fieldset. Django provides a "collapse" class that displays a particular fieldset initially collapsed. This is useful when you have a long form that contains a number of fields that aren't commonly used:

class PollAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]

Adding related objects

OK, we have our Poll admin page. But a Poll has multiple Choices, and the admin page doesn't display choices.

Yet.

There are two ways to solve this problem. The first is to register Choice with the admin just as we did with Poll. That's easy:

from polls.models import Choice

admin.site.register(Choice)



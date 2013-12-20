=================
Misc
=================


settings.py and localsettings.py
------------------------------------
The settings for your project which are a machine specific should be refactored
out of settings.py into localsettings.py. In your settings.py, you should do::

    try:
        from localsettings import *
    except ImportError:
        print 'localsettings could not be imported'
        pass #Or raise

This should be at the end of settings.py, so that localsetting.py override
settings in settings.py

This file should not be checked in your repository.


Use relative path in settings.py
--------------------------------------
Instead of writing::

    TEMPLATE_DIRS = '/home/user/project/templates'

Do::

    #settings.py
    import os

    CURRENT_DIR = os.path.dirname(__file__)
    TEMPLATE_DIRS = os.path.join(CURRENT_DIR, 'template')


Apps should provide default values for settings they are trying to read.
---------------------------------------------------------------------------
As far as possible, apps should have defaults for settings they are trying to
read. Instead of::

    DEFAULT_SORT_UP = settings.DEFAULT_SORT_UP

Use::

    DEFAULT_SORT_UP = getattr(settings, 'DEFAULT_SORT_UP' , '&uarr;')



Use templatetag when the output does not depend on the request
-------------------------------------------------------------------
In the sidebar, you want to show the 5 latest comments. You do not need
the request to output this. Make it a templatetag.


Import as if your apps are on your project path
----------------------------------------------------
Instead of doing `from project.app.models import ModelClass` do `from app.models
import ModelClass`. This makes you apps reusable as they are not tied to a project.

Naming things
-----------------

Model class names should be singular, not plural.::

    class Post(models.Model):
        ...

and not::

    class Posts(models.Model):
        ...

Foreign key should use the name of the referenced class.::

    class Post(models.Model):
        user = models.ForeignKey(User)

Querysets should be plural, instances should be singular.::

    posts = Post.objects.all()
    posts = Post.objects.filter(...)

    post = Post.object.get(pk = 5)
    post = Post.object.latest()

Using pdb remotely
------------------------
Sometimes you will hit bugs which show up on server but not on your local
system. To handle these, you need to debug on the server. Doing `manage.py
runserver` only allows local connections. To allow remote connections, use::

    python manage.py runserver 0.0.0.0:8000

So that your `pdb.set_trace()` which are on remote servers are hit when you access
them from your local system.


Do not use primary keys in urls
-----------------------------------
If you use PK in urls you are giving away sensitive information, for example,
the number of entries in your table. It also makes it trivial to guess other urls.

Use slugs in urls. This has the advantage of being both user and SEO
friendly.

If slugs do not make sense, instead use a CRC algorithm.::

    class Customer(models.Model):
        name = models.CharField(max_length = 100)

        def get_absolute_url(self):
            import zlib
            #Use permalink in real case
            return '/customer/%s/' % zlib.crc32(self.pk)


Code defensively in middleware and context processors.
-----------------------------------------------------------

Your middleware and context processors are going to be run for **all** requests.
Have you handled all cases?

    def process_request(request):
        if user.is_authenticated():
            profile = request.user.get_profile()
            # Hah, I create profiles during
            # registration so this is safe.
            ...


Or it is? What about users created via `manage.py createsuperuser`? With the
above middleware, the default user can not access even the admin site.

Hence handle all scenarios in middleware and context processors. This is one place
where `try: .. except: ..` (bare except) blocks are acceptable. You do not want one
middleware  bringing down the entire site.


Move long running tasks to a message queue.
------------------------------------------------
If you have long running requests they should be handled in a message queue, and not in the request thread. For example, using a lot of API calls, will make your pages crawl. Instead move the API processing to a message queue such as `celery <http://celeryproject.org/>`_.


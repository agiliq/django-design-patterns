=================
Urls
=================

Projects and apps
--------------------

There should be one `urls.py` at the project level, and one `urls.py` at each app
level. The project level `urls.py` should include each of the `urls.py` under a
prefix.::

    #project urls.py
    
    urlpatterns = patterns(
        '',
        (r'^', include('mainpages.urls')),
        (r'^admin/', include(admin.site.urls)),
        (r'^captcha/', include('yacaptcha.urls')),
        .....
    )
    
    #app urls.py
    urlpatterns = patterns(
        'app.views',
        url(r'^$', 'index'),
        url(r'^what/$', 'what_view')
        .....
    )

Naming urls
---------------

Urlpatterns should be named. [#ref1]_ This is done as::

    url(r'^$', 'index', name='main_index'),
    
This enables calling `{% url urlpatternname %}` much easier.

The pattern name should be of the form `appname_viewname`. If the same view is
used in multiple urlpatterns, the name should be of form `appname_viewname_use`,
as in `search_advanced_product` and `search_advanced_content`.::

    #urls.py for app search
    urlpatterns = patterns(
    'search.views'
    url(r'^advanced_product_search/$', 'advanced', name='search_advanced_product'),
    url(r'^advanced_content_search/$', 'advanced', name='search_advanced_content'),
    ...
    )
    
Here the same view `advanced` is used at two different urls and has two different names.

References
----------------

.. [#ref1] http://github.com/agiliq/django-blogango/blob/9525dfa621ca54219eed0c0e9c1624de89948045/blogango/urls.py#L23

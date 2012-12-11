=================
Templates
=================

Projects and apps.
--------------------
There should be one base.html at the project level, and one `base.html` at each of
the app levels. The app level `base.html` should extend the project level
`base.html`.::

    {# Eg Project base.html #}
    
    <html>
    <head>
    <title>{% block title %}My Super project{% endblock %}</title>
    ...
    
    {# app base.html #}
    
    {% extends 'base.html' %}
    
    {% block title %}{{ block.super }} - My duper app {% endblock %}
    ...
    
    
    {# login.html #}
    
    {% extends 'auth/base.html' %}
    {% block title %}{{ block.super }} - Login {% endblock %}
    ...
    

Location of templates
----------------------------

The templates for an app should be available as `appname/template.html`. So the
templates should be physically located at either

1. project/templates/app/template.html
2. project/app/templates/app/template.html

This allows two apps to have the same templates names.

Handling iterables which maybe empty
-----------------------------------------

In your view you do::

    posts = BlogPosts.objects.all()
    ...
    payload = {'posts': posts}
    return render_to_response('blog/posts.html', payload, ...)
    
Now `posts` may be empty, so in template we do,::

    <ul>    
    {% for post in posts %}
        <li>{{ post.title }}</li>
        {% empty %}
        <li>Sorry, no posts yet!</li>
    {% endfor %}
    <ul>
    
Please, note about `empty` clause using. If `posts` is empty or could not be found, the `empty` clause will be displayed.

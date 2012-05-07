=================
Views
=================

Generic views
------------------
Use generic views where possible. 

Generic views are just functions
------------------------------------
This means you can use them instead of calling say, `render_to_response`. For
example, suppose you want to show a list of objects, so you would like to use
`django.views.generic.object_list`. However, you also want to allow comments to
be posted on these objects, which this generic view does not allow. [#ref1]_ ::

    def object_list_comment(request):
        if request.method == 'POST':
            form = CommentForm(request.POST)
            if form.is_valid():
                obj = form.save()
                ...
                #redirect
        #Handle get or invalid form Post
        queryset = ModelClass.object.filter(...)
        payload = {'form':form}
        return object_list(request, queryset, extra_context = payload)
    

Handle GET and POST in same view function
----------------------------------------------

This keeps things grouped logically together. [#ref2]_ Eg.::

    def foo(request):
        form = FormClass()
        if request.method == 'POST':
            #Handle POST and form saving etc.
            #Redirect etc
        #Any more GET handling
        payload = {'form': form, ...}
        return render_to_response(...)


Querysets are chainable and lazy
-----------------------------------
This means that your view can keep on creating querysets and they would be
evaluated only when used. Suppose you have an advanced search view which
can take multiple criteria all of which are optional.::

    def advanced_search(request, criteria1=None, criteria2=None, criteria3=None):
        queryset = ModelClass.objects.all()
        if criteria1:
            queryset = queryset.filter(critera1=critera1)
        if criteria2:
            queryset = queryset.filter(critera2=critera2)
        if criteria3:
            queryset = queryset.filter(critera3=critera3)
        return objects_list(request, queryset=queryset)

References
----------------

.. [#ref1] http://github.com/mightylemon/mightylemon/blob/ff916fec3099d0edab5ba7b07f4cf838ba6fec7b/apps/events/views.py
.. [#ref2] http://github.com/agiliq/django-blogango/blob/9525dfa621ca54219eed0c0e9c1624de89948045/blogango/views.py#L65
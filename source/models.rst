=================
Models
=================

Multiple managers
--------------------
A Model class can have multiple managers, depending upon your needs. Suppose you
do not want to display any object on your site which is unapproved(is_approved =
False in your Model).::

    class ModelClassApprovedOnlyManager(models.Manager):
        def get_query_set(*args, **kwargs):
            return super(ModelClassApprovedOnlyManager, self).get_query_set(*args, **kwargs).filter(is_approved = True)
    
    class ModelClass(models.Model):
        ...
        is_approved = models.BooleanField(default = False)
        
        objects = models.Manager()
        approved_objects = ModelClassApprovedOnlyManager()
        
If you use multiple managers, the first manager should be the default manager. This is as the first
manager is accessible as `ModelClass._default_manager`, which is used by admin to get all objects.

Custom Manager Methods
----------------------
Imagine you have a query like this::
    
    Event.objects.filter(is_published=True).filter(start_date__gte=datetime.datetime.now()).order_by('start_date')

you probably will need to filter by status and created date again, to avoid duplicating 
code you could add custom methods to your default manager::

    class EventQuerySet(models.query.QuerySet):
        def published(self):
            return self.filter(is_published=True)
        
        def upcoming(self):
            return self.filter(start_date__gte=datetime.datetime.now())        
    
    class EventManager(models.Manager):
        def get_query_set(self):
            return EventQuerySet(self.model, using=self._db) # note the `using` parameter, new in 1.2
        
        def published(self):
            return self.get_query_set().published()
        
        def upcoming(self):
            return self.get_query_set().upcoming()
    
    class Event(models.Model):
        is_published = models.BooleanField(default=False)
        start_date = models.DateTimeField()
        ...
        
        objects = EventManager()    # override the default manager
    

This way you keep your logic in your model.
Why do you need a custom QuerySet? To be able to chain method calls. Now that query could be::

    Event.objects.published().upcoming().order_by('start_date')

Hierarchical Relationships
----------------------------
You may want to model hierarchical relationships. The simplest way to do this is::

    class ModelClass(models.Model):
        ...
        parent = models.ForeignKey('ModelClass')
        
This is called adjacency list model, and is very inefficient for large trees. If your
trees are very shallow you can use this. Otherwise you want to use a more
efficient but complex modeling called MPTT. Fortunately, you can just use django-mptt.

Singleton classes
-------------------
Sometimes you want to make sure that only one Object of a Model can be created.

Logging
-----------
To make sure, when an object is create/edited/deleted, there is a log.

Audit Trail and rollback
----------------------------
When an object is modified or deleted, to be able to go back to the previous
version.

Define an __unicode___
--------------------------
Until you define an `__unicode__` for your ModelClass, in Admin and at various
other places you will get an `<ModelClass object>` where the object needs to be
displayed. Define a meaningful `__unicode__` for you ModelClass, to get
meaningful display. Once you define `__unicode__`, you do not need to define
`__str__`.

Define a get_absolute_url()
-----------------------------
`get_absolute_url` is used at various places by Django. (In Admin for "view on
site" option, and in feeds framework).

Use reverse() for calculating get_absolute_url
---------------------------------------------------
You want only one canonical representation of your urls. This should be in urls.py

The `permalink` decorator is `no longer recommended <https://docs.djangoproject.com/en/1.5/ref/models/instances/#the-permalink-decorator>`_ for use.

If you write a class like::

    class Customer(models.Model)
        ...
        
        def get_absolute_url(self):
            return /customer/%s/ % self.slug

You have this representation at two places. You instead want to do::

    class Customer(models.Model)
        ...
        
        def get_absolute_url(self):
            return reverse('customers.detail', args=[self.slug])

AuditFields
----------------

You want to keep track of when an object was created and updated. Create
two DateTimeFields with `auto_now` and `auto_now_add`.::

    class ItemSold(models.Model):
        name = models.CharField(max_length = 100)
        value = models.PositiveIntegerField()
        ...
        #Audit field
        created_on = models.DateTimeField(auto_now_add = True)
        updated_on = models.DateTimeField(auto_now = True)
        
Now you want, created_by and updated_by. This is possible using the
`threadlocals <http://code.djangoproject.com/wiki/CookBookThreadlocalsAndUser>`_
technique, but since we `do not want <http://www.b-list.org/weblog/2008/dec/24/admin/>`_
to do that, we will need to pass user to the methods.::

    class ItemSoldManager(models.Manager):
        def create_item_sold(self, user, ...):
            

    class ItemSold(models.Model):
        name = models.CharField(max_length = 100)
        value = models.PositiveIntegerField()
        ...
        #Audit field
        created_on = models.DateTimeField(auto_now_add = True)
        updated_on = models.DateTimeField(auto_now = True)
        created_by = models.ForeignKey(User, ...)
        updated_by = models.ForeignKey(User, ...)
        
        def set_name(self, user, value):
            self.created_by = user
            self.name = value
            self.save()
            
        ...

    objects = ItemSoldManager()
    
Working with denormalised fields
-----------------------------------

Working with child tables.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You want to keep track of number of employees of a department.::

    class Department(models.Model):
        name = models.CharField(max_length = 100)
        employee_count = models.PositiveIntegerField(default = 0)
        
        
    class Employee(models.Model):
        department = models.ForeignKey(Department)
    
One way to do so would be to override, `save` and `delete`.::

    class Employee(models.Model):
        ...
        
        def save(self, *args, **kwargs):
            if not self.id:
                #this is a create, not an update
                self.department.employee_count += 1
                self.department.save()
            super(Employee, self).save(*args, **kwargs)
            
        def delete(self):
            self.department.employee_count -= 1
            self.department.save()
            super(Employee, self).delete()
            
Other option would be to attach listeners for `post_save` and `post_delete`.::

    from django.db.models import signals
    
    def increment_employee_count(sender, instance, created, raw, **kwargs):
        if created:
            instance.department.employee_count += 1
            instance.department.save()
    
    def decrement_employee_count(sender, instance, **kwargs):
        instance.department.employee_count -= 1
        instance.department.save()
    
    signals.post_save.connect(increment_employee_count, sender=Employee)
    signals.post_delete.connect(decrement_employee_count, sender=Employee)
    
    
Abstract custom queries in Manager methods.
----------------------------------------------

If you have some complex Sql query, not easily representable via Django ORM,
you can write custom Sql. These should be abstracted as Manager methods.

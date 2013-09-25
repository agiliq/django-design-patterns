=================
Forms
=================

Prefer ModelForm to Form
--------------------------
ModelForm already know the correct UI widgets for your underlying Models. In
most of the cases ModelForm would suffice instead of Forms.

Some common scenarios

Hiding some fields from ModelForm which are needed for a DB save.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Eg, you want to create a profile for the logged in user.::

    #in models.py
    class Profile(models.Model):
        user = models.OneToOneField(User)
        company = models.CharField(max_length=50)

    #in forms.py
    class ProfileForm(forms.ModelForm):
        class Meta:
            model = Profile
            fields = ['company',]
            
    #In views.py:
    form = ProfileForm(request.POST)
    profile = form.save(commit = False)
    profile.user = request.user
    profile.save()

Or::

    class ProfileForm(forms.ModelForm):

        class Meta:
            model = Profile
            fields =['company',]

        def __init__(self, user, *args, **kwargs)
            self.user = user
            super(ProfileForm, self).__init__(*args, **kwargs)
            
        def save(self, *args, **kwargs):
            self.instance.user = self.user
            super(ProfileForm, self).save(*args, **kwargs)


Customizing widgets in ModelForm fields
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sometimes you just need to override the widget of a field that's already on 
your ModelForm. Instead of duplicating the field definition (with `help_text`, 
`required`, `max_length`, etc). You can do this::
 
    from django.contrib.admin.widgets import AdminFileWidget

    class ProfileForm(forms.ModelForm):
        class Meta:
            model = Profile
            fields = ['picture', 'company']
    
        def __init__(self, *args, **kwargs):
            super(ProfileForm, self).__init__(*args, **kwargs)
            # note that self.fields is available just after calling super's __init__
            self.fields['picture'].widget = AdminFileWidget()


Saving multiple Objects in one form
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As::

    class ProfileForm(forms.ModelForm):
        class Meta:
            model = Profile
            fields = ['company',]
            
    class UserForm(forms.ModelForm):
        class Meta:
            model = User
            fields = [...]
            
    #in views.py
    userform = UserForm(request.POST)
    profileform =  ProfileForm(request.POST)
    if userform.is_valid() and profileform.is_valid():
        #Only if both are valid together
        user = userform.save()
        profile = profileform.save(commit = False)
        profile.user = user
        profile.save()
        
    {# In templates #}
    <form ...>
    {{ userform }}
    {{ profileform }}
    <input type="submit" />
    </form>
    


    
Forms should know how to save themselves.
---------------------------------------------

If your forms is a `forms.ModelForm`, it already knows how to save its data. If you
write a forms.Form, it should have a `.save()`. This keeps things symmetrical with
`ModelForms`, and allows you to do::

    #in views.py
    def view_func(request):
        if request.method == 'POST':
            form  = FormClass(request.POST)
            if form.is_valid():
                obj = form.save()
                ...
            ...

Instead of::

            if form.is_valid():
                #handle the saving in DB inside of views.
                
The `.save()` should return a Model Object


The form should know what to do with it's data
------------------------------------------------

If you're building a contact form, or something like this, the goal of your form is
to send an email. So this logic should stay in the form::

    class ContactForm(forms.Form):
        subject = forms.CharField(...)
        message = forms.TextField(...)
        email = forms.EmailField(...)
        ...
        
        def save(self):
            mail_admins(self.cleaned_data['subject'], self.cleaned_data['message'])

I've used `save()`, and not `send()`, even when i'm not really saving anything. 
This is just a convention, people prefer to use `save()` to keep the same interface to
ModelForms. But it doesn't really matter, call it whatever you want.

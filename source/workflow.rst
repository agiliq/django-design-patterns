=================
Workflow
=================

Use a source control system
-------------------------------
Use SVN, GIT, Hg whatever. But choose one and use it.

Create a requirements.txt
----------------------------------
Your project should have a requirements.txt file. A `pip install -r requirements.txt`
should get all the third party apps which are not part of your source control system.

Use virtualenv and pip 
---------------------------
Your various projects might require different versions of third party libraries. Use `virtualenv <http://pypi.python.org/pypi/virtualenv>`_ to keep
separate environments and use `pip <http://www.pip-installer.org/en/latest/index.html>`_ to manage dependencies.

Use pep8.py to check compliance with Python coding guidelines.
----------------------------------------------------------------
Your code would be using conforming to pep8, which are the standard coding guidelines. `Pep8.py <http://pypi.python.org/pypi/pep8>`_ can check your code for deviations.


Use pyflakes for static analysis.
----------------------------------------------------------------
`Pyflakes <http://pypi.python.org/pypi/pyflakes>`_ can find out some common mistakes. Run it after each deploy.


Use a bug tracking tool.
----------------------------
I recommend `Unfuddle <http://unfuddle.com/>`_, (It has various niceties, above
a source control and bug tracking tool).
But others might work for you. In particular Trac is free.

Use south for schema migration
---------------------------------
Django doesn't come with a built in tool for Schema Migration. `South <http://south.aeracode.org/>`_ is the best tool for managing schema migrations and is well supported.

Create various entries in your /etc/hosts mapped to localhost
------------------------------------------------------------------
While development you probably want multiple users logged in to the site
simultaneously. For example, while developing, I have one user logged in the
admin, one normal
user using the site. If both try to access the site from localhost, one will be
logged out when other logs in.

If you have multiple entries mapped to localhost in /etc/hosts, you can use
multiple users simultaneously logged in.

Do not commit the generated files
-----------------------------------
Django does not have a lot of auto generated files. However as you work with
other django apps, you may come across auto generated files. These should not be
checked in the the Django repository.
For example, for this book, we checkin the source files and folder, but not the
auto-generated build folders.




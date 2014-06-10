Notes on maintaining python environments
========================================

.. _python_environment:

Normal setup 
-------------

Install `Python 2.7.3  <http://www.python.org/ftp/python/2.7.3/python-2.7.3.amd64.msi>`_

Run an Admin PowerShell Enviromment and Run::

  Set-ExecutionPolicy RemoteSigned

(this is a little scary, but basically means it won't run anything
that you download from the internet that is not signed)

Select *y*.

Then, from a normal powershell prompt, cd into your copy of the atlas code::

  cd \
  cd git\atlas

Then::

  ./activate-python.ps1

And hit *A* to always run the script.  From here you can run ipython using the command::

  ipython

Installing your own python environment
--------------------------------------

To find the packages you need, activate the PYTHON27 virtual
environment and run::

  pip list

Many the goodies you need are here:: 

  K:\Research\Python (\\tmrfp02\analysis$\research\python)

First, install python itself: python-2.7.3.amd64.msi  

Now, install all the packages listed by ``pip``.

Pure python packages will install without problems.  However, any
that use C-extensions will probably fail with ``pip``.  Configuring a
machine so that it can build C-extensions is challenging.  You
currently need Visual Studio 2008 and to have it configured to be able
to build 64-bit applications. 

Fortunately, there is an excellent site which has pre-built binary
packages you can install: http://www.lfd.uci.edu/~gohlke/pythonlibs/

Look for the ``amd64-2.7`` versions, i.e. 64-bit for python 2.7.

Adding binary packages to virtualenv python
-------------------------------------------

Note to install msi/exe stuff into a virtualenv you have to do some
hackery.  The problem is these things look in the registry to find
your python, but the virtualenv pythons are not registered.

To get round this there is a script, ``python/scripts/register.py`` that
will register your current python.

So the trick is, switch to the virtualenv python (using activate) then
run the register script::

  python python/scripts/register.py

Then install stuff.  Finally, deactivate your virtual python and
re-run the registry script with normal python and you will be good.

You will need to use a powershell with administrator priveleges to do
the registry changes.

Making sure virtualenv is relocatable
-------------------------------------

By default, virtualenv hard-wires the path of the environment into the
various scripts.  To make it so others can take a copy and put it
where they want you need to run::

  virtualenv --relocatable PYTHON27

Further, this needs to be done each time you add a new package to the
environment.

Other miscellaneous configuration
---------------------------------

Graphviz and pygraphviz
~~~~~~~~~~~~~~~~~~~~~~~

  - download and install the msi for graphviz from here: http://www.graphviz.org/Download_windows.php
  - add the path for the graphviz binaries to your system Path "C:\Program Files (x86)\Graphviz2.32\bin" 
  
pygraphviz currently has a bug relating to the location of the graphviz
executables: it fails to properly quote the path to the executable
and so you get some error about "C:\Program" not existing.

This is fixed in the development version, but meanwhile you need to
fix Python27/Lib/site-packages/pygraphviz/agraph.py around line 1250
to add this line to properly quote runprog::

  runprog=r'"%s"'%self._get_prog(prog)

Addendum: fixed in pygraphviz 1.2.


Configuring django to talk to SQL Server
----------------------------------------

Install binary pyodbc package.

Use pip to install ``django-pydobc-azure``::

  pip install django-pydobc-azure

See: ``UI.settings.production`` and ``UI.settings.sqlexpress`` for how to
configure the djano database connection to connect to SQL server.

Running django apps under IIS
-----------------------------

WFastCGI
~~~~~~~~

First install *WFastCGI 2.0 Gateway for IIS and Python 2.7*

There is done using Web Platform Installer (WPI).

* Fire up WPI.
* Enter *CGI* in the search box and it should show you the cgi related
  modules that are available
* Add *WFastCGI 2.0 Gateway for IIS and Python 2.7*
* Click *Install*


This will probably fail with a message about Python 2.7 not being
installed.

The problem is the installer is looking for 32-bit python 2.7 and we
are using 64-bit.

The fix is to install 32-bit python.  Put it in C:/Python27_32 and
make sure it is registered as the default python.  This can be done by
running::

  C:\Python27_32\python  <GIT>python/scripts/register.py

After this you should be able to do the WFastCGI install.

Next you need to run::

  C:\Python27\python   <GIT>python/scripts/register.py

to switch back to 64-bit python.

Configuring IIS to run django
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is done using `django windows tools
<https://pypi.python.org/pypi/django-windows-tools>`_.

To install the django apps in IIS you just need to run::

  python manage.py winfcgi_install --settings UI.settings.production

Note that this needs to be done in a powershell with Administrator
priveleges.

If you run it in a normal powershell you will get a confusing message
saying::

  CGI does not seem to be installed

You will then waste the next few hours trying to figure out how to
install the WFastCGI module in IIS when it is in fact already
installed.

Once you have run this, in theory everything should just work.  In
practice, if you go to a URL that your app should be serviing you will
probably just get the browser hanging.

It is best to be running the browser locally on the IIS machine at
this point as it gives you a little more help as to what is going on.

If things are just hanging the issue is probably that IIS cannot
access the files it is trying to serve.  Check the Application Pool
permissions as well as permissions for individual handler mappings.

Once this is all done, try loading a page.  Odds are you will get a
*500 internal server error*.  IIS doesn not give you much more to go
on than this, especially if the winfcgi process is dying before it
gets off the ground.  The best way to see what is going on is to run:: 

  python manage.py winfcgi --settings UI.settings.production

and hope it spits out some error such as a missing import.

Serving static content
~~~~~~~~~~~~~~~~~~~~~~

A good test of the new site is to try and serve static content.  You
will actually need to do this anyway so the django application can get
at css, javascript and other static content.

Since the WFastCGI handler is set up with a wildcard to match all
URL's you need to place the static content handlers ahead of this.

Right-click in the Handler Mappints and select *View ordered list* and
ensure the WFastCGI handler comes last.

To add static handlers use "Add module Mapping".  Select
*StaticFileHandler* as the the Module and enter ``*.css`` as the request
path and name it *css*.

You will need to add a handler for each file suffix that you need to
serve (eg .js, .png, .gif etc).  The dialog looks like you can have
one handler and a comma-separated list of wildcards, but this does not
work. 

Finally, you should click *Request Restrictions..* and under the
*Mapping* tab select the check box for *Invoke the handler only if the
request is mapped to*  and select *File or Folder* from the radio
buttons. 


Serving documentation and other static content
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To create a shortcut URL to serve documentation you can just add a
virtual directory to the site.

Select the path, *inetpub/wwwroot/atlas/docs*.  The handlers for
the main site should be inherited and it should just work.

To serve static content (javascript, css) associated with the django
site you need to first run::

   python manage.py collectstatic --settings UI.settings.production

This will create a folder, *static* that should be copied to
*inetpub/wwwroot/atlas/statoc*::

  robocopy /MIR  static inetpub/wwwroot/atlas/static


Using windows authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Django REMOTE_USER authentication
<https://docs.djangoproject.com/en/dev/howto/auth-remote-user/>`_ is
the way to get this working.

It is simply a matter of adding the
``django.contrib.auth.middleware.RemoteUserMiddleware`` to settings
and specifying RemoteUserBackend as an authentication backend::

  AUTHENTICATION_BACKENDS = (
      'django.contrib.auth.backends.RemoteUserBackend',
  )


In fact, to make this work smoothly AtlasRemoteAuth is derived from RemoteUserBackend.

.. autoclass:: UI.auth.AtlasRemoteAuth
   :members:

This allows us to remove the domain name from user names before
entering them in the django user database.  This is necessary since
*\\* is not a valid character in django user names.

It also allows us to do some configuration for users when they log
on.  Currently, this is just used to add them to the
*portfolio_manager* group so that they get enough permissions to be
dangerous. 

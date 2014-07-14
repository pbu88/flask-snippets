Apache hosting
==============

I had problems with the default Apache wsgi-setup from the Flask docs:
`http://flask.pocoo.org/docs/deploying/mod_wsgi/#creating-a-wsgi-
file`_

I simply changed the python current-working-directory in the wsgi-file
to the application directory.

Now Apache is fine.

myapp.wsgi:


::

    import sys, os
    sys.path.insert (0,'/var/www/myapp')
    os.chdir("/var/www/myapp")
    from srv import app as application
.. _http://flask.pocoo.org/docs/deploying/mod_wsgi/#creating-a-wsgi-file: http://flask.pocoo.org/docs/deploying/mod_wsgi/#creating-a-wsgi-file


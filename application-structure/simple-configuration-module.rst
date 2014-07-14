Simple Configuration Module
===========================

When it comes to structuring your application the question about a
configuration comes up. The easiest way to use a configuration files
is to import a configuration module.

For example you could have a file called `websiteconfig.py` that looks
like this:


::

    DEBUG = False
    SECRET_KEY = 'mysecretkey'
    DATABASE_URI = 'sqlite:////tmp/myapp.db


Then you could use it in your application as follows:


::

    import websiteconfig
    app.debug = websiteconfig.DEBUG
    app.secret_key = websiteconfig.SECRET_KEY


If you are using packages like outlined in the `documentation`_ you
can put something like this into your `__init__.py`:


::

    import websiteconfig as config


Then you can import the config like this in any module:


::

    from yourapplication import config


This makes it very easy to later change the name of the config
filename, to load different configuration modules based on environment
or to switch the module to an INI file later.
.. _documentation: http://flask.pocoo.org/docs/patterns/packages/


Simple Configuration Module
===========================

When it comes to structuring your application the question about a configuration comes up. The easiest way to use a configuration files is to import a configuration module.

For example you could have a file called ``websiteconfig.py`` that looks like this:

.. code-block:: python

    DEBUG = False
    SECRET_KEY = 'mysecretkey'
    DATABASE_URI = 'sqlite:////tmp/myapp.db

Then you could use it in your application as follows:

.. code-block:: python

    import websiteconfig
    app.debug = websiteconfig.DEBUG
    app.secret_key = websiteconfig.SECRET_KEY

If you are using packages like outlined in the documentation_ you can put something like this into your ``__init__.py``:

.. code-block:: python

    import websiteconfig as config

Then you can import the config like this in any module:

.. code-block:: python

    from yourapplication import config

This makes it very easy to later change the name of the config filename, to load different configuration modules based on environment or to switch the module to an INI file later.

**Alternatively**, Another way is to always use the same name for your config module, but to have a "local_config" imported into the module:

.. code-block:: python

    # config.py

    DATABASE_URI = "sqlite://"

    try:
        from local_config import *
    except ImportError:
        # no local config found
        pass

That way you can keep all your defaults the same, but provide overridden values as needed. The local_config module presumably would not be in your public repository.

.. _documentation: http://flask.pocoo.org/docs/patterns/packages/
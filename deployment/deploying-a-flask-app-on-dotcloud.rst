Deploying a Flask app on Dotcloud
=================================

*Please note: this snippet does not cover the connection to the
database.*


How to deploy a Flask app on Dotcloud
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Create the namespace you want:

::

    dotcloud create <namespace>


A DotCloud application is described by a `build file`_, which is a
simple YAML file named "dotcloud.yml" located in your local source
directory. To add a new python service to your app, just add the
following lines to `<source_folder>/dotcloud.yml`:

::

    www:
      type: python


Now create a `<source_folder>/wsgi.py` file containing:


::

    import sys
    sys.path.append('/home/dotcloud/current')
    from <your_app_package> import app as application


`/home/dotcloud/current` is the default path to your app on the
server.

Eventually, create `nginx.conf` and `uwsgi.conf` in the source folder.
See `Dotcloud documentation`_ for further information.

Create your `requirements.txt` file:

::

    pip freeze > ./requirements.txt


Now, make sure that your source folder contains at least:

::

    ./
    ../
    ./<your_app_package>
    	./static
    	./__init__.py
    	./ ...
    ./requirements.txt
    ./wsgi.py
    ./dotcloud.yml
    ./ ...


Create a symbolic link to your `static` folder:

::

    cd <source_folder>
    ln -s <your_app_package>/static static


You can now push the code to Dotcloud:

::

    dotcloud push <namespace>


A random URL has been generated for your python service in your
application (something like `http://my4ppr0x.dotcloud.com/`_). Point
your browser to this URL to see your new app running.
.. _Dotcloud documentation: http://docs.dotcloud.com/services/python/
.. _build file: http://docs.dotcloud.com/guides/build-file/
.. _http://my4ppr0x.dotcloud.com/: http://my4ppr0x.dotcloud.com/


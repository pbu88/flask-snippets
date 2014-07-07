Deploy using zc.buildout and PythonPaste
========================================

Foreword
--------
There are two common ways to develop Python applications:

- virtualenv_
- `zc.buildout`_

This article describes how to use **zc.buildout** to develop, deploy and run a Flask application.

In addition it features some **pythonpaste** utilities:

- Paste_ for the WSGI HTTP server (with a thread pool).
- PasteDeploy_ for the WSGI server (and logging) configuration.
- PasteScript_ to serve the application (command **bin/paster**).

Outstanding features:

- central configuration for application and server: *buildout.cfg*
- easy and repeatable deployment
- run the application with different configurations (development, production)
- run the server as a daemon (*nix only)
- run the test suite using nose_

Create the buildout environment
-------------------------------
The buildout directory should look like this:

.. code-block:: none

    +-buildout_env/
        +-bootstrap.py
        +-buildout.cfg
        +-etc/
        |   +-deploy.ini.in
        +-README
        +-setup.py
        +-src/
            +-hello/
                +-__init__.py
                +-script.py
                +-tests.py

First create the directory structure

.. code-block:: bash

    ~ $ mkdir buildout_env
    ~ $ cd buildout_env
    ~/buildout_env $ mkdir -p etc src/hello

Then download the `bootstrap.py`_ file in the buildout directory.

Edit the **src/hello/__init__.py** file:

.. code-block:: python

    # -*- coding: utf-8 -*-
    from flask import Flask, request

    class _DefaultSettings(object):
        USERNAME = 'world'
        SECRET_KEY = 'development key'
        DEBUG = True

    # create the application
    app = Flask(__name__)
    app.config.from_object(_DefaultSettings)
    del _DefaultSettings


    def init_db():
        """Create the database tables."""
        pass


    @app.route('/')
    def index():
        if request.args:
            BREAK (with_NameError)
        return 'Hello %s!' % app.config['USERNAME'].title()

Edit the **src/hello/tests.py** file:

.. code-block:: python

    # -*- coding: utf-8 -*-
    import unittest
    import hello


    class HelloTestCase(unittest.TestCase):

        def setUp(self):
            """Before each test, set up a blank database"""
            self.app = hello.app.test_client()
            hello.init_db()

        def tearDown(self):
            """Get rid of the database again after each test."""
            pass

        def test_hello(self):
            """Test rendered page."""
            hello.app.config['USERNAME'] = 'jean'
            rv = self.app.get('/')
            assert 'Hello Jean!' in rv.data


    def suite():
        suite = unittest.TestSuite()
        suite.addTest(unittest.makeSuite(HelloTestCase))
        return suite


    if __name__ == '__main__':
        unittest.main()

Edit the **src/hello/script.py** file:

.. code-block:: python

    # -*- coding: utf-8 -*-
    """Startup utilities"""
    import os
    import sys
    from functools import partial

    import paste.script.command
    import werkzeug.script

    etc = partial(os.path.join, 'parts', 'etc')

    DEPLOY_INI = etc('deploy.ini')
    DEPLOY_CFG = etc('deploy.cfg')

    DEBUG_INI = etc('debug.ini')
    DEBUG_CFG = etc('debug.cfg')

    _buildout_path = __file__
    for i in range(2 + __name__.count('.')):
        _buildout_path = os.path.dirname(_buildout_path)

    abspath = partial(os.path.join, _buildout_path)
    del _buildout_path


    # bin/paster serve parts/etc/deploy.ini
    def make_app(global_conf={}, config=DEPLOY_CFG, debug=False):
        from hello import app
        app.config.from_pyfile(abspath(config))
        app.debug = debug
        return app


    # bin/paster serve parts/etc/debug.ini
    def make_debug(global_conf={}, **conf):
        from werkzeug.debug import DebuggedApplication
        app = make_app(global_conf, config=DEBUG_CFG, debug=True)
        return DebuggedApplication(app, evalex=True)


    # bin/flask-ctl shell
    def make_shell():
        """Interactive Flask Shell"""
        from flask import request
        from hello import init_db as initdb
        app = make_app()
        http = app.test_client()
        reqctx = app.test_request_context
        return locals()


    def _init_db(debug=False, dry_run=False):
        """Initialize the database."""
        from hello import init_db
        print 'init_db()'
        if dry_run:
            return
        # Configure the application
        if debug:
            make_debug()
        else:
            make_app()
        # Create the tables
        init_db()


    def _serve(action, debug=False, dry_run=False):
        """Build paster command from 'action' and 'debug' flag."""
        if action == 'initdb':
            # First, create the tables
            return _init_db(debug=debug, dry_run=dry_run)
        if debug:
            config = DEBUG_INI
        else:
            config = DEPLOY_INI
        argv = ['bin/paster', 'serve', config]
        if action in ('start', 'restart'):
            argv += [action, '--daemon']
        elif action in ('', 'fg', 'foreground'):
            argv += ['--reload']
        else:
            argv += [action]
        # Print the 'paster' command
        print ' '.join(argv)
        if dry_run:
            return
        # Configure logging and lock file
        if action in ('start', 'stop', 'restart', 'status'):
            argv += [
                '--log-file', abspath('var', 'log', 'paster.log'),
                '--pid-file', abspath('var', 'log', '.paster.pid'),
            ]
        sys.argv = argv[:2] + [abspath(config)] + argv[3:]
        # Run the 'paster' command
        paste.script.command.run()


    # bin/flask-ctl ...
    def run():
        action_shell = werkzeug.script.make_shell(make_shell, make_shell.__doc__)

        # bin/flask-ctl serve [fg|start|stop|restart|status|initdb]
        def action_serve(action=('a', 'start'), dry_run=False):
            """Serve the application.

            This command serves a web application that uses a paste.deploy
            configuration file for the server and application.

            Options:
             - 'action' is one of [fg|start|stop|restart|status|initdb]
             - '--dry-run' print the paster command and exit
            """
            _serve(action, debug=False, dry_run=dry_run)

        # bin/flask-ctl debug [fg|start|stop|restart|status|initdb]
        def action_debug(action=('a', 'start'), dry_run=False):
            """Serve the debugging application."""
            _serve(action, debug=True, dry_run=dry_run)

        # bin/flask-ctl status
        def action_status(dry_run=False):
            """Status of the application."""
            _serve('status', dry_run=dry_run)

        # bin/flask-ctl stop
        def action_stop(dry_run=False):
            """Stop the application."""
            _serve('stop', dry_run=dry_run)

        werkzeug.script.run()

Create the **README** file:

.. code-block:: none

                         / hello /

                "Hello World!" application 

Edit the **setup.py** file:

.. code-block:: python

    from setuptools import setup, find_packages
    import os

    name = "hello"
    version = "0.1"


    def read(*rnames):
        return open(os.path.join(os.path.dirname(__file__), *rnames)).read()


    setup(
        name=name,
        version=version,
        description="a hello world demo",
        long_description=read('README'),
        # Get strings from http://www.python.org/pypi?%3Aaction=list_classifiers
        classifiers=[],
        keywords="",
        author="",
        author_email='',
        url='',
        license='',
        package_dir={'': 'src'},
        packages=find_packages('src'),
        include_package_data=True,
        zip_safe=False,
        install_requires=[
            'setuptools',
            'Flask',
        ],
        entry_points="""
        [console_scripts]
        flask-ctl = hello.script:run

        [paste.app_factory]
        main = hello.script:make_app
        debug = hello.script:make_debug
        """,
    )

Edit the **etc/deploy.ini.in** file:

.. code-block:: none

    # ${:outfile}
    #
    # Configuration for use with paster/WSGI
    #

    [loggers]
    keys = root, wsgi

    [handlers]
    keys = console, accesslog

    [formatters]
    keys = generic, accesslog

    [formatter_generic]
    format = %(asctime)s %(levelname)s [%(name)s] %(message)s

    [formatter_accesslog]
    format = %(message)s

    [handler_console]
    class = StreamHandler
    args = (sys.stderr,)
    level = NOTSET
    formatter = generic

    [handler_accesslog]
    class = FileHandler
    args = (os.path.join(r'${server:logfiles}', 'access.log'), 'a')
    level = INFO
    formatter = accesslog

    [logger_root]
    level = INFO
    handlers = console

    [logger_wsgi]
    level = INFO
    handlers = accesslog
    qualname = wsgi
    propagate = 0

    [filter:translogger]
    use = egg:Paste#translogger
    setup_console_handler = False
    logger_name = wsgi

    [app:main]
    use = egg:${:app}
    filter-with = translogger

    [server:main]
    use = egg:Paste#http
    host = ${server:host}
    port = ${server:port}
    threadpool_workers = ${:workers}
    threadpool_spawn_if_under = ${:spawn_if_under}
    threadpool_max_requests = ${:max_requests}

Edit the **buildout.cfg** file:

.. code-block:: none

    [buildout]
    develop = .
    parts =
        app
        mkdirs
        deploy_ini
        deploy_cfg
        debug_ini
        debug_cfg
        test
    newest = false

    # eggs will be installed in the default buildout location
    # (see .buildout/default.cfg in your home directory)
    # unless you specify an eggs-directory option here.

    [server]
    host = 127.0.0.1
    port = 5000
    logfiles = ${buildout:directory}/var/log

    [app]
    recipe = zc.recipe.egg
    eggs = hello
           Paste
           PasteScript
           PasteDeploy

    interpreter = python-console

    [mkdirs]
    recipe = z3c.recipe.mkdir
    paths =
        ${server:logfiles}

    [deploy_ini]
    recipe = collective.recipe.template
    input = etc/deploy.ini.in
    output = ${buildout:parts-directory}/etc/${:outfile}
    outfile = deploy.ini
    app = hello
    workers = 10
    spawn_if_under = 5
    max_requests = 100

    [debug_ini]
    <= deploy_ini
    outfile = debug.ini
    app = hello#debug
    workers = 1
    spawn_if_under = 1
    max_requests = 0

    [deploy_cfg]
    recipe = collective.recipe.template
    input = inline:
        # Deployment configuration
        DEBUG = False
        SECRET_KEY = 'production key'
        USERNAME = 'Fernand'
    output = ${buildout:parts-directory}/etc/deploy.cfg

    [debug_cfg]
    recipe = collective.recipe.template
    input = inline:
        # Debugging configuration
        DEBUG = True
        SECRET_KEY = 'development key'
        USERNAME = 'Raoul'
    output = ${buildout:parts-directory}/etc/debug.cfg

    [test]
    recipe = pbp.recipe.noserunner
    eggs = hello
    defaults = -v

Deploy the application
----------------------
First, you could save the buildout directory using your favorite DVCS, or create a tarball for future deployments.

Then bootstrap the buildout:

.. code-block:: bash

    ~/buildout_env $ python bootstrap.py --distribute

Adjust your settings in **buildout.cfg**, and build the application:

.. code-block:: bash

    ~/buildout_env $ bin/buildout

Run the tests:

.. code-block:: bash

    ~/buildout_env $ bin/test
    Test rendered page. ... ok

    ------------------------------------------------------------
    Ran 1 test in 0.055s

    OK
    ~/buildout_env $ 

Now launch the server:

.. code-block:: bash

    ~/buildout_env $ bin/flask-ctl debug fg
    bin/paster serve parts/etc/debug.ini --reload
    Starting subprocess with file monitor
    Starting server in PID 24862.
    serving on http://127.0.0.1:5000

Visit http://127.0.0.1:5000 with your browser.

Visit http://127.0.0.1:5000/?broken to bring the `Werkzeug Debugger`_. Quit the application with *Ctrl+C*.

Note: when you change the configuration in **buildout.cfg**, you need to rebuild the application using **bin/buildout**.

Further reading:

- http://www.buildout.org
- http://pythonpaste.org

**Comment:**
The multi-threading capability of the Paste server is not compatible with Werkzeug Debugger. I've added settings to buildout.cfg to run a single thread in debug mode, and never kill the thread. The production mode is not affected. (See threadpool options 'workers', 'spawn_if_under' and 'max_requests')


*This snippet can be used freely for anything you like. Consider it public domain.*

.. _virtualenv: http://flask.pocoo.org/docs/installation/#virtualenv
.. _zc.buildout: http://www.buildout.org/
.. _Paste: http://pypi.python.org/pypi/Paste
.. _PasteDeploy: http://pypi.python.org/pypi/PasteDeploy
.. _PasteScript: http://pypi.python.org/pypi/PasteScript
.. _nose: http://somethingaboutorange.com/mrl/projects/nose/
.. _bootstrap.py: http://svn.zope.org/*checkout*/zc.buildout/trunk/bootstrap/bootstrap.py
.. _Werkzeug Debugger: http://werkzeug.pocoo.org/documentation/dev/debug.html#using-the-debugger
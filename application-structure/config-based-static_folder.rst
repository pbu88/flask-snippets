Config-based static_folder
==========================

A response to a `question on the mailing list`_: how replace the `static_folder parameter`_ by a config value?

.. code-block:: python

    import flask
    class MyFlask(flask.Flask):

        @property
        def static_folder(self):
            if self.config.get('STATIC_FOLDER') is not None:
                return os.path.join(self.root_path, 
                    self.config.get('STATIC_FOLDER'))

        @static_folder.setter
        def static_folder(self, value):
            self.config.get('STATIC_FOLDER') = value

    # Now these are equivalent:
    app = Flask(__name__, static_folder='foo')

    app = MyFlask(__name__)
    app.config['STATIC_FOLDER'] = 'foo'

However since the URL rule is still created in __init__ this only work for setting a different path, not for disabling it completely with None.

.. code-block:: python

    # Still no static URL rule, /static/foo.png gives HTTP 404
    app = MyFlask(__name__, static_folder=None)
    app.config['STATIC_FOLDER'] = 'foo'

    # /static/foo.png gives HTTP 500 rather than 404:
    app = Flask(__name__)
    app.config['STATIC_FOLDER'] = None

.. _question on the mailing list: http://flask.pocoo.org/mailinglist/archive/2012/10/5/change-static-folder-from-configuration-file/
.. _static_folder parameter: http://flask.pocoo.org/docs/api/#flask.Flask
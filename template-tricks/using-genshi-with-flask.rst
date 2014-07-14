Using Genshi with Flask
=======================

Jinja is a very nice text-based templating engine, but if you want an
XML-based templating engine, Genshi is a good choice.

`Flask-Genshi`_ is a Flask extension that makes it easy. First install
it:


::

    $ easy_install Flask-Genshi


Here's a simple app:


::

    from flask import Flask
    from flaskext.genshi import Genshi, render_response
    
    app = Flask(__name__)
    genshi = Genshi(app)
    
    @app.route('/')
    def index():
        render_response('index.html')


For documentation on Genshi see `the Genshi wiki`_.
.. _the Genshi wiki: http://genshi.edgewall.org/wiki/Documentation
.. _Flask-Genshi: http://packages.python.org/Flask-Genshi/


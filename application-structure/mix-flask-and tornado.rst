Mix Flask and Tornado together
==============================

This is how you load a Flask application using Tornado and mix Tornado + Flask together.

First here is *flasky.py* the file where the flask application is:

.. code-block:: python 

    from flask import Flask
    app = Flask(__name__)

    @app.route('/flask')
    def hello_world():
      return 'This comes from Flask ^_^'

and then the *cyclone.py* the file which will load the flask application and the tornado server + a simple tornado application, hope there is no module called "cyclone" ^_^

.. code-block:: python

    from tornado.wsgi import WSGIContainer
    from tornado.ioloop import IOLoop
    from tornado.web import FallbackHandler, RequestHandler, Application
    from flasky import app

    class MainHandler(RequestHandler):
      def get(self):
        self.write("This message comes from Tornado ^_^")

    tr = WSGIContainer(app)

    application = Application([
    (r"/tornado", MainHandler),
    (r".*", FallbackHandler, dict(fallback=tr)),
    ])

    if __name__ == "__main__":
      application.listen(8000)
      IOLoop.instance().start()

based of `this stackoverflow answer`_


.. _this stackoverflow answer: http://stackoverflow.com/questions/8143141/using-flask-and-tornado-together/8247457#8247457
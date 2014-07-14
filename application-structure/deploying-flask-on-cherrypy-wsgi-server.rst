Deploying Flask on Cherrypy WSGI server
=======================================

`CherryPy`_ comes with a `WSGI compliant server`_, so running a Flask
application on top of CherryPy is a piece of cake.

Here is the Flask hello world app for reference:


::

    from flask import Flask
    app = Flask(__name__)
    
    @app.route("/")
    def hello():
        return "Hello World!"
    
    if __name__ == "__main__":
        app.run()


The following snippet let you run the Flask app on top of the WSGI
server shipped with CherryPy.


::

    from cherrypy import wsgiserver
    from hello import app
    
    d = wsgiserver.WSGIPathInfoDispatcher({'/': app})
    server = wsgiserver.CherryPyWSGIServer(('0.0.0.0', 8080), d)
    
    if __name__ == '__main__':
       try:
          server.start()
       except KeyboardInterrupt:
          server.stop()


You can access the server running on 0.0.0.0:8080.
WSGIPathInfoDispatcher take as argument a dictionary mapping a path to
an application object, so you can easily deploy multiple (Flask)
application on a single CherryPy server.
.. _CherryPy: http://www.cherrypy.org
.. _http://fgimian.github.io/blog/2012/12/08/setting-up-a-rock-solid-python-development-web-server/: http://fgimian.github.io/blog/2012/12/08/setting-up-a-rock-solid-python-development-web-server/
.. _WSGI compliant server: http://www.cherrypy.org/wiki/WSGI


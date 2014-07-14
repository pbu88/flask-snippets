Reloading with other WSGI servers
=================================

If you want to use reloading with a different WSGI server, you can use
`werkzeug.serving.run_with_reloader` directly.

For example, this snippet uses one of `gevent's WSGIServers`_:


::

    import gevent.wsgi
    import werkzeug.serving
    
    @werkzeug.serving.run_with_reloader
    def runServer():
        app.debug = True
    
        ws = gevent.wsgi.WSGIServer(('', 5000), app)
        ws.serve_forever()
.. _http://hastebin.com/luduhexiso.py: http://hastebin.com/luduhexiso.py
.. _http://werkzeug.pocoo.org/docs/debug/: http://werkzeug.pocoo.org/docs/debug/
.. _https://github.com/aldanor/SocketIO-Flask-Debug: https://github.com/aldanor/SocketIO-Flask-Debug
.. _gevent's WSGIServers: http://www.gevent.org/servers.html


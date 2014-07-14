Passing REMOTE_USER from Apache as a reverse proxy to web application
=======
-------
By Armin Ronacher filed in `Deployment`_
The `REMOTE_USER` key in the CGI/WSGI environment is well established
as to where servers should put the identity of an authenticated user.
This is also where mod_wsgi will store it. But what if you are using
HTTP proxying? If you are using mod_proxy in Apache you can use the
`RequestHeader` directive to pass that information to Flask as an HTTP
header:


::

    RequestHeader set X-Proxy-REMOTE-USER %{REMOTE_USER}


Then you only need a small WSGI middleware to rename the key:


::

    class RemoteUserMiddleware(object):
        def __init__(self, app):
            self.app = app
        def __call__(self, environ, start_response):
            user = environ.pop('HTTP_X_PROXY_REMOTE_USER', None)
            environ['REMOTE_USER'] = user
            return self.app(environ, start_response)


And hook in the middleware:


::

    app.wsgi_app = RemoteUserMiddleware(app.wsgi_app)


Inside a view you can then access `REMOTE_USER` as usual:


::

    remote_user = request.environ.get('REMOTE_USER')
    if remote_user:
        ...


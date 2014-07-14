Overriding HTTP Methods for old browsers
========================================

HTML4 and XHTML only specify POST and GET as HTTP methods that forms
can use. HTTP itself however supports a wider range of methods, and it
makes sense to support them on the server.

If you however want to make a form submission with PUT for instance,
and you are using a client that does not support it, you can override
it.

First you need to hook this middleware in:


::

    from werkzeug import url_decode
    
    class MethodRewriteMiddleware(object):
    
        def __init__(self, app):
            self.app = app
    
        def __call__(self, environ, start_response):
            if 'METHOD_OVERRIDE' in environ.get('QUERY_STRING', ''):
                args = url_decode(environ['QUERY_STRING'])
                method = args.get('__METHOD_OVERRIDE__')
                if method:
                    method = method.encode('ascii', 'replace')
                    environ['REQUEST_METHOD'] = method
            return self.app(environ, start_response)


To then override the method, you have to append
`?__METHOD_OVERRIDE__=PUT` to the form action:


::

    <form action="?__METHOD_OVERRIDE__=PUT">
      ...
    </form>
.. _http://flask.pocoo.org/docs/quickstart/#hooking-in-wsgi-middlewares: http://flask.pocoo.org/docs/quickstart/#hooking-in-wsgi-middlewares


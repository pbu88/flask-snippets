Supporting “;” as Delimiter in Legacy Query Strings
===================================================

If you have an application that accepted “;” as alternative to “&” in
query strings in the past and you want to continue to support these
URLs you can hook in this little WSGI middleware to rewrite them on
the fly before passing them over to Flask:


::

    class QueryStringRedirectMiddleware(object):
    
        def __init__(self, application):
            self.application = application
    
        def __call__(self, environ, start_response):
            qs = environ.get('QUERY_STRING', '')
            environ['QUERY_STRING'] = qs.replace(';', '&')
            return self.application(environ, start_response)


To activate it, just wrap the `wsgi_app` attribute:


::

    app.wsgi_app = QueryStringRedirectMiddleware(app.wsgi_app)


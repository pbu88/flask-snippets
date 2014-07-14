Cherokee fix for URL prefix
===========================

The snippet below is required when hosting a flask application using
Cherokee+uWSGI with a prefixed URL.


::

    class CherrokeeFix(object):
    
        def __init__(self, app, script_name):
            self.app = app
            self.script_name = script_name
    
        def __call__(self, environ, start_response):
            path = environ.get('SCRIPT_NAME', '') + environ.get('PATH_INFO', '')
            environ['SCRIPT_NAME'] = self.script_name
            environ['PATH_INFO'] = path[len(self.script_name):]
            assert path[:len(self.script_name)] == self.script_name
            return self.app(environ, start_response)


If this URL was prefixed with '/test', you would apply this to the
Flask application as below:


::

    app.wsgi_app = CherrokeeFix(app.wsgi_app, '/test')


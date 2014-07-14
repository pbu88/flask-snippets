Adding caching to Flask apps
============================

Werkzeug includes a number of caching options in the
werkzeug.contrib.cache package: `werkzeug.contrib.cache`_

However the API just handles low-level caching - this is handy for
caching individual objects (such as the results of a database query)
but not if you want to cache an entire view, or even a whole site.

For a per-view cache we can use a decorator:


::

    from werkzeug.contrib.cache import SimpleCache
    
    CACHE_TIMEOUT = 300
    
    cache = SimpleCache()
    
    class cached(object):
    
        def __init__(self, timeout=None):
            self.timeout = timeout or CACHE_TIMEOUT
    
        def __call__(self, f):
            def decorator(*args, **kwargs):
                response = cache.get(request.path)
                if response is None:
                    response = f(*args, **kwargs)
                    cache.set(request.path, response, self.timeout)
                return response
            return decorator
    
    @app.route("/")
    @cached()
    def index():
        return render_template("index.html")


This example uses SimpleCache, for production you probably want to use
memcached.

To cache your whole application, you can use before_request and
after_request:


::

    @app.before_request
    def return_cached():
        # if GET and POST not empty
        if not request.values:
            response = cache.get(request.path)
            if response: 
                return response
    
    @app.after_request
    def cache_response(response):
        if not request.values:
            cache.set(request.path, response, CACHE_TIMEOUT)
        return response


We ignore the cache if the GET or POST are not empty.

You might want to modify these functions further; for example, you
might want to ignore the cache in certain views by setting an
"ignore_cache" flag in the request or g objects, or if the user is
authenticated.

For template-level caching in Jinja2, see this example: `Cache Example
Extension`_
.. _http://flask.pocoo.org/docs/patterns/viewdecorators/: http://flask.pocoo.org/docs/patterns/viewdecorators/
.. _Flask-Cache extension: http://packages.python.org/Flask-Cache/
.. _Cache Example Extension: http://jinja.pocoo.org/2/documentation/extensions#example-extension
.. _werkzeug.contrib.cache: http://werkzeug.pocoo.org/documentation/dev/contrib/cache.html
.. _http://flask.pocoo.org/docs/patterns/caching/: http://flask.pocoo.org/docs/patterns/caching/


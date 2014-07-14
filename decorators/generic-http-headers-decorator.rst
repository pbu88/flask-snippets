Generic HTTP headers decorator
==============================

If you need to add response header(s) to a view, this
`add_response_headers` decorator does that - pass it a dict of header
names and values.

You can wrap the decorator to make convenience decorators for headers
you need to add often, as shown by the `no index` decorator, which
adds `X-Robots-Tag: noindex` to the response.


::

    from functools import wraps
    
    from flask import Flask, make_response
    
    app = Flask(__name__)
    
    
    def add_response_headers(headers={}):
        """This decorator adds the headers passed in to the response"""
        def decorator(f):
            @wraps(f)
            def decorated_function(*args, **kwargs):
                resp = make_response(f(*args, **kwargs))
                h = resp.headers
                for header, value in headers.items():
                    h[header] = value
                return resp
            return decorated_function
        return decorator
    
    
    def noindex(f):
        """This decorator passes X-Robots-Tag: noindex"""
        @wraps(f)
        @add_response_headers({'X-Robots-Tag': 'noindex'})
        def decorated_function(*args, **kwargs):
            return f(*args, **kwargs)
        return decorated_function
    
    
    @app.route('/')
    @noindex
    def not_indexed():
        """
        This page will be served with X-Robots-Tag: noindex
        in the response headers
        """
        return "Check my headers!"
    
    if __name__ == "__main__":
        app.run(host='0.0.0.0')
    
    # check the headers with: curl -I http://0.0.0.0:5000/


Content negotiated error responses
==================================

If you would like to trigger an HTTP exception like `abort` in Flask
does, but prefer application/json this is one way to do it. You can
raise with any JSON serializable object. If the user agent is a
browser it will fall back to the Werkzeug errors which are HTML
formatted.

Usage


::

    from .exceptions import abort
    
    @app.route("/test")
    def view():
        abort(422, {'errors': dict(password="Wrong password")})


Snippet


::

    from werkzeug.exceptions import default_exceptions, HTTPException
    from flask import make_response, abort as flask_abort, request
    from flask.exceptions import JSONHTTPException
    
    
    def abort(status_code, body=None, headers={}):
        """
        Content negiate the error response.
    
        """
    
        if 'text/html' in request.headers.get("Accept", ""):
            error_cls = HTTPException
        else:
            error_cls = JSONHTTPException
    
        class_name = error_cls.__name__
        bases = [error_cls]
        attributes = {'code': status_code}
    
        if status_code in default_exceptions:
            # Mixin the Werkzeug exception
            bases.insert(0, default_exceptions[status_code])
    
        error_cls = type(class_name, tuple(bases), attributes)
        flask_abort(make_response(error_cls(body), status_code, headers))


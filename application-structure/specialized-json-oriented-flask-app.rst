Specialized JSON-oriented Flask App
===================================

Let's say you are creating a RESTful web service that typically sees JSON requests and responds with JSON back. When things go wrong, default errors that Flask/Werkzeug respond with are all HTML. Which breaks the clients who expect JSON back even in case of errors.

Here's an approach to mitigate this. All errors that Werkzeug may throw are now intercepted and converted into JSON response. You can customize what goes into the response by tweaking the line with ``response = jsonify(...)``.

Also note that ``make_json_error`` will be used when your code throws an arbitrary exception (e.g. division by zero) not derived from ``HTTPException``.

.. code-block:: python

    from flask import Flask, jsonify
    from werkzeug.exceptions import default_exceptions
    from werkzeug.exceptions import HTTPException

    __all__ = ['make_json_app']

    def make_json_app(import_name, **kwargs):
        """
        Creates a JSON-oriented Flask app.

        All error responses that you don't specifically
        manage yourself will have application/json content
        type, and will contain JSON like this (just an example):

        { "message": "405: Method Not Allowed" }
        """
        def make_json_error(ex):
            response = jsonify(message=str(ex))
            response.status_code = (ex.code
                                    if isinstance(ex, HTTPException)
                                    else 500)
            return response

        app = Flask(import_name, **kwargs)

        for code in default_exceptions.iterkeys():
            app.error_handler_spec[None][code] = make_json_error

        return app


*This snippet can be used freely for anything you like. Consider it public domain.*
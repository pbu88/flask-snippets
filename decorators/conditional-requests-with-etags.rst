Conditional Requests with ETags
===============================

Here we demonstrate conditional request execution using the If-Match
and If-None-Match headers with a monkeypatched Werkzeug method,
triggered by a decorator in Flask. This technique is flexible about
how and when we calculate ETag s.


Decorator
~~~~~~~~~

First let's examine the decorator. This is really simple: it just sets
a flag on the flask.g object before every invocation of the view
method. In a previous version of this decorator we had created an
object at this level on each invocation, but that was before I really
grokked the Tao of g .


::

    def conditional(func):
        '''Start conditional method execution for this resource'''
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            flask.g.condtnl_etags_start = True
            return func(*args, **kwargs)
        return wrapper




Monkeypatch
~~~~~~~~~~~

By monkeypatching the set_etag() method, we run the conditional logic
exactly when it needs to be run, but only in view functions that are
decorated by @conditional , and only the first time that set_etag() is
called in those decorated functions.


::

    _old_set_etag = werkzeug.ETagResponseMixin.set_etag
    @functools.wraps(werkzeug.ETagResponseMixin.set_etag)
    def _new_set_etag(self, etag, weak=False):
        # only check the first time through; when called twice
        # we're modifying
        if (hasattr(flask.g, 'condtnl_etags_start') and
                                   flask.g.condtnl_etags_start):
            if flask.request.method in ('PUT', 'DELETE', 'PATCH'):
                if not flask.request.if_match:
                    raise PreconditionRequired
                if etag not in flask.request.if_match:
                    flask.abort(412)
            elif (flask.request.method == 'GET' and
                  flask.request.if_none_match and
                  etag in flask.request.if_none_match):
                raise NotModified
            flask.g.condtnl_etags_start = False
        _old_set_etag(self, etag, weak)
    werkzeug.ETagResponseMixin.set_etag = _new_set_etag




Application
~~~~~~~~~~~

Here's how we can use these tools in an application. After decorating
with @conditional , the normal set_etag() method will check the
calculated etag against the conditional headers. Since the PUT and
PATCH verbs change a resource, set_etag() is called again after the
change to provide an updated ETag on the ensuing 204 No Content
response.


::

    app = flask.Flask(__name__)
    d = {'a': 'This is "a".\n', 'b': 'This is "b".\n'}
    @app.route('/<path>',
               methods = ['GET', 'PUT', 'DELETE', 'PATCH'])
    @conditional
    def view(path):
        try:
            # SHA1 should generate well-behaved etags
            etag = hashlib.sha1(d[path]).hexdigest()
            if flask.request.method == 'GET':
                response = flask.make_response(d[path])
                response.set_etag(etag)
            else:
                response = flask.Response(status=204)
                del response.headers['content-type']
                response.set_etag(etag)
                if flask.request.method == 'DELETE':
                    del d[path]
                    del response.headers['etag']
                else:
                    if flask.request.method == 'PUT':
                        d[path] = flask.request.data
                    else: # (PATCH)
                        # lame PATCH technique
                        d[path] += flask.request.data
                 response.set_etag(hashlib.sha1(d[path])
                                          .hexdigest())
            return response
        except KeyError:
            flask.abort(404)
    app.run()




Testing with curl
~~~~~~~~~~~~~~~~~

The behavior seems correct. It's the server's option whether or not to
force conditional updates with the brand new 428 Precondition Required
status as we've done here.

::

    $ curl -i localhost:5000/a
    HTTP/1.0 200 OK
    Content-Type: text/html; charset=utf-8
    Content-Length: 13
    ETag: "56eaadbbd9fa287e7270cf13a41083c94f52ab9b"
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Sat, 07 Jul 2012 00:45:03 GMT
    
    This is "a".
    
    $ curl -iH 'If-None-Match: \
    "56eaadbbd9fa287e7270cf13a41083c94f52ab9b"' localhost:5000/a
    HTTP/1.0 304 NOT MODIFIED
    Connection: close
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Sat, 07 Jul 2012 00:45:12 GMT
    
    $ curl -iX DELETE localhost:5000/a
    HTTP/1.0 428 PRECONDITION REQUIRED
    Content-Type: text/html
    Content-Length: 214
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Sat, 07 Jul 2012 00:45:19 GMT
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <title>428 Precondition Required</title>
    <h1>Precondition Required</h1>
    <p>This request is required to be conditional; try using
    "If-Match".
    
    $ curl -iX DELETE -H 'If-Match: "badmatch"' localhost:5000/a 
    HTTP/1.0 412 PRECONDITION FAILED
    Content-Type: text/html
    Content-Length: 203
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Sat, 07 Jul 2012 00:45:23 GMT
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <title>412 Precondition Failed</title>
    <h1>Precondition Failed</h1>
    <p>The precondition on the request for the URL failed positive
    evaluation.</p>
    
    $ curl -iX DELETE -H 'If-Match: \
    "56eaadbbd9fa287e7270cf13a41083c94f52ab9b"' localhost:5000/a
    HTTP/1.0 204 NO CONTENT
    Content-Length: 0
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Sat, 07 Jul 2012 00:45:30 GMT
    
    $ curl -i localhost:5000/a
    HTTP/1.0 404 NOT FOUND
    Content-Type: text/html
    Content-Length: 238
    Server: Werkzeug/0.8.3 Python/2.7.3
    Date: Sat, 07 Jul 2012 00:45:35 GMT
    
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <title>404 Not Found</title>
    <h1>Not Found</h1>
    <p>The requested URL was not found on the server.</p><p>If you
    entered the URL manually please check your spelling and try
    again.</p>


...and so on like that! PUT and PATCH are handled similarly to DELETE
.


Weren't there some exceptions?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are two custom HTTP status exceptions. 428 Precondition Required
was just introduced in `RFC 6585`_. 304 Not Modified isn't an error
per se, but it's handy to use an exception for a short-circuit
response like this.


::

    class NotModified(werkzeug.exceptions.HTTPException):
        code = 304
        def get_response(self, environment):
            return flask.Response(status=304)
    
    class PreconditionRequired(werkzeug.exceptions.HTTPException):
        code = 428
        description = ('<p>This request is required to be '
                       'conditional; try using "If-Match".')
        name = 'Precondition Required'
        def get_response(self, environment):
            resp = super(PreconditionRequired,
                         self).get_response(environment)
            resp.status = str(self.code) + ' ' + self.name.upper()
            return resp


Suggestions for improvement are very welcome!
.. _RFC 6585: http://www.rfc-editor.org/rfc/rfc6585.txt


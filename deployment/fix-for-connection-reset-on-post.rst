Fix for Connection Reset on POST
================================

You might notice that if you start not accessing `.form` or `.files`
on incoming POST requests, some browsers will honor this with a
connection reset message. This can happen if you start rejecting
uploads that are larger than a given size.

Some WSGI servers solve that problem for you, others do not. For
instance the builtin Flask webserver is pretty dumb and will not
attempt to fix this problem.

Thankfully it's easy to fix this in a WSGI middleware. But first we
have to figure out why it's actually happening. The reason this
happens is that the browser will have queued up quite some bytes sent
and will not start reading from the socket unless it transmitted
everything to the server. Because the server however is not accepting
the data, it's not responding.

The fix is to actually read all the bytes on the server and discarding
them.

This can do the trick for you:


::

    from werkzeug.wsgi import LimitedStream
    
    
    class StreamConsumingMiddleware(object):
    
        def __init__(self, app):
            self.app = app
    
        def __call__(self, environ, start_response):
            stream = LimitedStream(environ['wsgi.input'],
                                   int(environ['CONTENT_LENGTH'] or 0))
            environ['wsgi.input'] = stream
            app_iter = self.app(environ, start_response)
            try:
                stream.exhaust()
                for event in app_iter:
                    yield event
            finally:
                if hasattr(app_iter, 'close'):
                    app_iter.close()


To apply the middleware in Flask, do this:


::

    app.wsgi_app = StreamConsumingMiddleware(app.wsgi_app)


What does this do? We wrap the stream in a `LimitedStream`. This class
in Werkzeug internally keeps track of how many bytes were consumed and
will not read past the given limit (The `CONTENT_LEGNTH`). The call to
`exhaust()` will start exhausting all bytes after the application
provided the response. The downside of this solution is that your
application must not not read bytes in a streamed response.


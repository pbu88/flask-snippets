Gzip for particular views
=========================

Normally, the task of gzipping responses before sending them to
clients is handled by web servers, like apache or nginx. In such
cases, the web server acts as a middleman between client's request and
your app's response.

However on some environments it is not possible to deploy with such
'middleman'. One of example of that environment is Heroku: even their
documentation says your app should be connected directly to clients
via Gunicorn. Therefore, gzipping has to be implemented on your app's
end.

Below is code that I shamelessly modified from
`https://github.com/wichitacode/flask-compress`_, which is a great
extension but unfortunately applys gzipping to entire response. What I
needed was gzipping for some particular views.


::

    from flask import after_this_request, request
    from cStringIO import StringIO as IO
    import gzip
    import functools 
    
    def gzipped(f):
        @functools.wraps(f)
        def view_func(*args, **kwargs):
            @after_this_request
            def zipper(response):
                accept_encoding = request.headers.get('Accept-Encoding', '')
    
                if 'gzip' not in accept_encoding.lower():
                    return response
    
                response.direct_passthrough = False
    
                if (response.status_code < 200 or
                    response.status_code >= 300 or
                    'Content-Encoding' in response.headers):
                    return response
                gzip_buffer = IO()
                gzip_file = gzip.GzipFile(mode='wb', 
                                          fileobj=gzip_buffer)
                gzip_file.write(response.data)
                gzip_file.close()
    
                response.data = gzip_buffer.getvalue()
                response.headers['Content-Encoding'] = 'gzip'
                response.headers['Vary'] = 'Accept-Encoding'
                response.headers['Content-Length'] = len(response.data)
    
                return response
    
            return f(*args, **kwargs)
    
        return view_func


usage:


::

    @app.route('/')
    @gzipped
    def gzip_me():
      return "this response has to be gzipped"
.. _https://github.com/wichitacode/flask-compress: https://github.com/wichitacode/flask-compress


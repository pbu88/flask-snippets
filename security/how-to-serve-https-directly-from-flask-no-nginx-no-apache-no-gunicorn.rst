How to serve HTTPS *directly* from Flask (no nginx, no apache, no
=========
---------
By 0byte filed in `Security`_
This is a great tip for debugging your HTTPS-enabled application.

Create a SSL context (`http://werkzeug.pocoo.org/docs/serving/`_)

::

    from OpenSSL import SSL
    context = SSL.Context(SSL.SSLv23_METHOD)
    context.use_privatekey_file('yourserver.key')
    context.use_certificate_file('yourserver.crt')


then

::

    app.run(host='127.0.0.1',port='12344', 
            debug = False/True, ssl_context=context)


Linux-related:

there is a confirmed bug in pyOpenSSL that generates a runtime
error:`https://bugs.launchpad.net/pyopenssl/+bug/900792`_

The workaround is to put these 2 lines in werkzeug/serving.py

::

    in class BaseWSGIServer(HTTPServer, object):
    ...
     def shutdown_request(self,request):
            request.shutdown()


enjoy,

0byte
.. _https://bugs.launchpad.net/pyopenssl/+bug/900792: https://bugs.launchpad.net/pyopenssl/+bug/900792
.. _http://werkzeug.pocoo.org/docs/serving/: http://werkzeug.pocoo.org/docs/serving/


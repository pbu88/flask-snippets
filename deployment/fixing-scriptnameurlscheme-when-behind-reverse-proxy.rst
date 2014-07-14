Fixing SCRIPT_NAME/url_scheme when behind reverse proxy
=======================================================

When running behind a reverse proxy, not only is the public URL
different, but often you want the application to appear below some
path other than /. Sometimes you even use https from the outside, in
which case you likely don't use SSL between the proxy server and your
Flask app.

In such cases, if you can control the headers in the proxy server
(easy with Nginx and many others), this middleware lets you
transparently change where the application appears. *This works
dynamically for each request, so you can simultaneously access from
the local machine using `http://localhost:port` without using the full
URL.*


::

    class ReverseProxied(object):
        '''Wrap the application in this middleware and configure the 
        front-end server to add these headers, to let you quietly bind 
        this to a URL other than / and to an HTTP scheme that is 
        different than what is used locally.
    
        In nginx:
        location /myprefix {
            proxy_pass http://192.168.0.1:5001;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Scheme $scheme;
            proxy_set_header X-Script-Name /myprefix;
            }
    
        :param app: the WSGI application
        '''
        def __init__(self, app):
            self.app = app
    
        def __call__(self, environ, start_response):
            script_name = environ.get('HTTP_X_SCRIPT_NAME', '')
            if script_name:
                environ['SCRIPT_NAME'] = script_name
                path_info = environ['PATH_INFO']
                if path_info.startswith(script_name):
                    environ['PATH_INFO'] = path_info[len(script_name):]
    
            scheme = environ.get('HTTP_X_SCHEME', '')
            if scheme:
                environ['wsgi.url_scheme'] = scheme
            return self.app(environ, start_response)


Install in app using:


::

        app = Flask(__name__)
        app.wsgi_app = ReverseProxied(app.wsgi_app)


That's it! Now your page accessible locally at
`http://192.168.0.1:5001/myapp` will also be accessible externally at
`https://example.com/myprefix/myapp`.

Don't forget to `/etc/init.d/nginx reload` or the equivalent after
changing your server configuration.


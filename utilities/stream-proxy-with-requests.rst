Stream Proxy with Requests
==========================


::

    # -*- coding: utf-8 -*-
        
    from flask import Flask
    from flask import Response
    from flask import stream_with_context
    
    import requests
    
    app = Flask(__name__)
    
    @app.route('/<path:url>')
    def home(url):
        req = requests.get(url, stream = True)
        return Response(stream_with_context(req.iter_content()), content_type = req.headers['content-type'])
    
    if __name__ == '__main__':
        app.run()


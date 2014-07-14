Using Context Globals with Gevent-Socketio
==========================================

When using Gevent-Socketio, context globals are popped once the
original non-websocket request is closed, after `socketio_manage` is
called. This can be solved without any changes to Flask or Gevent-
Socketio.

First pass the real request into `socketio_manage`


::

    from socketio import socketio_manage
    @app.route('/socket.io/<path:path>')
    def run_socketio(path):
        real_request = request._get_current_object()
        socketio_manage(request.environ, {'': FlaskNamespace},
                request=real_request)
        return Response()


Now manually create and destroy contexts inside the namespace


::

    from socketio.namespace import BaseNamespace
    class FlaskNamespace(BaseNamespace):
        def __init__(self, *args, **kwargs):
            request = kwargs.get('request', None)
            self.ctx = None
            if request:
                self.ctx = current_app.request_context(request.environ)
                self.ctx.push()
                current_app.preprocess_request()
                del kwargs['request']
            super(BaseNamespace, self).__init__(*args, **kwargs)
    
        def disconnect(self, *args, **kwargs):
            if self.ctx:
                self.ctx.pop()
            super(BaseNamespace, self).disconnect(*args, **kwargs)


From here all of our `on_*` methods in `FlaskNamespace` will be able
to access `request`, `g` and other context globals.

One remaining issue is that stale sockets, or sockets that don't
disconnect when your application restarts, will raise an error on
disconnecting since their request does not exist. Once I get around to
finding a solution I will post it.


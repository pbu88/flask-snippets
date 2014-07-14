Shutdown The Simple Server
==========================

The Werkzeug server that is used by the `app.run()` command can be
shut down starting with Werkzeug 0.8. This can be helpful for small
applications that should serve as a frontend to a simple library on a
user's computer.


::

    from flask import request
    
    def shutdown_server():
        func = request.environ.get('werkzeug.server.shutdown')
        if func is None:
            raise RuntimeError('Not running with the Werkzeug Server')
        func()


Now you can shutdown the server by calling this function:


::

    @app.route('/shutdown', methods=['POST'])
    def shutdown():
        shutdown_server()
        return 'Server shutting down...'


The shutdown functionality is written in a way that the server will
finish handling the current request and then stop.


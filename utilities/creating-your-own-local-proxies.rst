Creating your own local proxies
===============================

Sometimes, you don't want to prefix whatever you are sharing in your
thread local with `g.` for some reason. To solve this problem, you can
create your own local-type proxy objects with:


::

    from werkzeug.local import LocalProxy
    
    whatever = LocalProxy(lambda: g.whatever)


Of course, this can be used for other things besides just `g`.


::

    from werkzeug.local import LocalProxy
    
    method = LocalProxy(lambda: request.method)


Now that Werkzeug 0.6.1 is out, it's really more of an AnythingProxy
than just a LocalProxy.


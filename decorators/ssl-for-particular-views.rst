SSL for particular views
========================

Sometime you want to enable https only for some of ulrs on your
website. For example, login, registration, checkout etc.

In this case you can use decorator like this:


::

    from functools import wraps
    from flask import request, redirect, current_app
    
    def ssl_required(fn):
        @wraps(fn)
        def decorated_view(*args, **kwargs):
            if current_app.config.get("SSL"):
                if request.is_secure:
                    return fn(*args, **kwargs)
                else:
                    return redirect(request.url.replace("http://", "https://"))
            
            return fn(*args, **kwargs)
                
        return decorated_view


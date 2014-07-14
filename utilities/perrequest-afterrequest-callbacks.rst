Per-Request After-Request Callbacks
===================================

Flask provides the `app.after_request` function to trigger the
execution of a function at the end of a request. This however is done
for all requests. Sometimes it can be useful to trigger some code that
modifies a response object only for one specific request.

For instance you might have some code that invalidates a cache
somewhere and wants to update a cookie at the end of a request.

This can be easily implemented by keeping a list of callbacks on the
`g` object:


::

    from flask import g
    
    def after_this_request(func):
        if not hasattr(g, 'call_after_request'):
            g.call_after_request = []
        g.call_after_request.append(func)
        return func
    
    
    @app.after_request
    def per_request_callbacks(response):
        for func in getattr(g, 'call_after_request', ()):
            response = func(response)
        return response


And here is how you can use it:


::

    def invalidate_username_cache():
        @after_this_request
        def delete_username_cookie(response):
            response.delete_cookie('username')
            return response


View Rendering Decorator
========================

If you are planning on providing your views in multiple formats (e.g.:
templated html, json, xml, whatever), you may like to return a dict
from your views, and use a function to render the response. This
snippet makes it easy to do that.

Usage:


::

    from helpers import view, render_html, render_json
    
    @view(app, '/<name>', render_html('page.html'))
    @view(app, '/api/page/<name>', render_json)
    def show_page(name):
        page = load_page(name)
        return dict(title=page.title, contents=page.contents)


The decorator to put in helpers.py is:


::

    from myapp import app
    from werkzeug import BaseResponse
    
    def render_html(template, **defaults):
        def wrapped(result):
            variables = defaults.copy()
            variables.update(result)
            return render_template(template, **variables)
        return wrapped
    
    def view(self, url, renderer=None, *args, **kwargs):
        super_route = self.route
    
        defaults = kwargs.pop('defaults', {})
        route_id = object()
        defaults['_route_id'] = route_id
    
        def deco(f):
            @super_route(url, defaults=defaults, *args, **kwargs)
            @wraps(f)
            def decorated_function(*args, **kwargs):
                this_route = kwargs.get('_route_id')
                if not getattr(f, 'is_route', False):
                    del kwargs['_route_id']
    
                result = f(*args, **kwargs)
    
                if this_route is not route_id:
                    return result
    
                # catch redirects.
                if isinstance(result, (app.response_class,
                                       BaseResponse)):
                    return result
    
                if renderer is None:
                    return result
                return renderer(result)
    
            decorated_function.is_route = True
            return decorated_function
    
        return deco


Other rendering functions (render_json, render_xml, etc.) are left as
an exercise for the reader. ;)


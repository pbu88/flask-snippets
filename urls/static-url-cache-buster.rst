static url cache buster
=======================

If you decide to add an expires header (and if you haven't already you
really should) to your static resources, you now need to worry about
cache busting these resources after your next deploy. A simple way of
dealing with this is to add a last modified query parameter to the end
of your resource. For example:


::

    <link rel="stylesheet" href="/static/css/reset.css?q=1280549780"
       type="text/css" media="screen" charset="utf-8" />


By adding the following snippet you can override the default
`url_for(endpoint, **values)` variable in your template context. Now
any time you use `url_for` in your templates to render a static
resource it will be appended with a last modified time stamp
parameter.


::

    @app.context_processor
    def override_url_for():
        return dict(url_for=dated_url_for)
    
    def dated_url_for(endpoint, **values):
        if endpoint == 'static':
            filename = values.get('filename', None)
            if filename:
                file_path = os.path.join(app.root_path,
                                         endpoint, filename)
                values['q'] = int(os.stat(file_path).st_mtime)
        return url_for(endpoint, **values)
.. _http://paste.pocoo.org/show/513299/: http://paste.pocoo.org/show/513299/


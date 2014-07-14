nl2br filter
============

This is a nl2br (newline to <BR>) filter, adapted from the Jinja2
example here:

`http://jinja.pocoo.org/2/documentation/api#custom-filters`_


::

    import re
    
    from jinja2 import evalcontextfilter, Markup, escape
    
    _paragraph_re = re.compile(r'(?:\r\n|\r|\n){2,}')
    
    app = Flask(__name__)
    
    @app.template_filter()
    @evalcontextfilter
    def nl2br(eval_ctx, value):
        result = u'\n\n'.join(u'<p>%s</p>' % p.replace('\n', '<br>\n') \
            for p in _paragraph_re.split(escape(value)))
        if eval_ctx.autoescape:
            result = Markup(result)
        return result
.. _http://jinja.pocoo.org/2/documentation/api#custom-filters: http://jinja.pocoo.org/2/documentation/api#custom-filters


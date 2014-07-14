Override which templates are autoescaped
========================================


::

    from flask import Flask
    
    class JHtmlEscapingFlask(Flask):
    
        def select_jinja_autoescape(self, filename):
            if filename.endswith('.jhtml'):
                return True
            return Flask.select_jinja_autoescape(self, filename)
    
    app = JHtmlEscapingFlask(__name__)


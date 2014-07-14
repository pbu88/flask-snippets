Using Babel's LazyProxy with gettext
====================================

Babel has a lazy proxy function under `babel.support`_, that you can
use when you need a lazy gettext string.

For example if you are defining field labels in a WTForms class:


::

    from wtforms import Form, fields
    from myapp.utils import ugettext_lazy as _
    
    class MyForm(Form):
        name = fields.TextField(_("Name"))


The string will be translated in the request thread rather than
immediately, as in most cases you will want to provide the translation
based on user settings (browser, session or database).


::

    from flask import g
    from babel.support import LazyProxy
    
    def ugettext(s):
        # we assume a before_request function
        # assigns the correct user-specific
        # translations
        return g.translations.ugettext(s)
    
    ugettext_lazy = LazyProxy(ugettext)
.. _babel.support: http://babel.edgewall.org/wiki/ApiDocs/babel.support
.. _speaklater: http://pypi.python.org/pypi/speaklater/1.1


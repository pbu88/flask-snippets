Preconfigured interactive shell
===============================

The documentation recommends putting imports in a module and import
that when you're in an interactive session. Even better, I think, is
to make an executable that launches a preconfigured Python shell.

Put something like this in for example `shell.py` and run `chmod +x
shell.py`.


::

    #!/usr/bin/env python
    
    import os
    import readline
    from pprint import pprint
    
    from flask import *
    
    from myapp import *
    from utils import *
    from db import *
    from models import *
    
    
    os.environ['PYTHONINSPECT'] = 'True'


Normally `import *` should be avoided but unless you get namespace
collisions it makes sense for an interactive shell environment.

Now you can simply do `./shell.py`.
.. _http://werkzeug.pocoo.org/documentation/0.6.1/script.html: http://werkzeug.pocoo.org/documentation/0.6.1/script.html
.. _Flask-Script: http://packages.python.org/Flask-Script/


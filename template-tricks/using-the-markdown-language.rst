Using the markdown language
===========================

The `Markdown language`_ is very useful if you don't know the html
language In this snippet, I want to show you how to include this
language in your Flask App

`The markdown library`_

::

    easy_install markdown


The code of your Flask application: demo.py


::

    # We import the markdown library
    import markdown
    from flask import Flask
    from flask import render_template
    from flask import Markup
    
    app = Flask(__name__)
    @app.route('/')
    
    def index():
      content = """
    Chapter
    =======
    
    Section
    -------
    
    * Item 1
    * Item 2
    """
      content = Markup(markdown.markdown(content))
      return render_template('index.html', **locals())
    
    app.run(debug=True)


The Jinja template file: templates/index.html


::

    <html>
      <head>
        <title>Markdown Snippet</title>
      </head>
      <body>
        {{ content }}
      </body>
    </html>
.. _The markdown library: http://www.freewisdom.org/projects/python-markdown/
.. _Markdown language: http://en.wikipedia.org/wiki/Markdown
.. _https://flask-misaka.readthedocs.org: https://flask-misaka.readthedocs.org
.. _http://packages.python.org/Flask-Markdown/: http://packages.python.org/Flask-Markdown/


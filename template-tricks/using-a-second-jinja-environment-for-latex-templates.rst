Using a second Jinja environment for LaTeX templates.
=====================================================

If you use your Flask application to primarily generate (X)HTML
content, but you also have views where you need to do some LaTeX, you
will soon run into a problem: Jinja’s default syntax doesn’t really
play well when creating LaTeX documents, as curly braces are an
integral part of LaTeX’s syntax as well.

Of course, you could change Jinja’s delimiters for this application.
You cannot do it for a single call to render_template() though, so
this means you’d have to change the markup in all (X)HTML templates as
well. As a consequence, this wouldn’t exactly leverage the re-use of a
template file in other project, e.g., one that contains only macros.

A more viable solution is to create a second Jinja environment with a
different set of delimiters. (Hail to EftarjinK for proposing this
solution on the Pocoo IRC channel!) Bonus feature: you can also create
a custom filter function that escapes LaTeX’s reserved characters for
you.


::

    app = Flask(__name__)
    
    LATEX_SUBS = (
        (re.compile(r'\\'), r'\\textbackslash'),
        (re.compile(r'([{}_#%&$])'), r'\\\1'),
        (re.compile(r'~'), r'\~{}'),
        (re.compile(r'\^'), r'\^{}'),
        (re.compile(r'"'), r"''"),
        (re.compile(r'\.\.\.+'), r'\\ldots'),
    )
    
    def escape_tex(value):
        newval = value
        for pattern, replacement in LATEX_SUBS:
            newval = pattern.sub(replacement, newval)
        return newval
    
    texenv = app.create_jinja_environment()
    texenv.block_start_string = '((*'
    texenv.block_end_string = '*))'
    texenv.variable_start_string = '((('
    texenv.variable_end_string = ')))'
    texenv.comment_start_string = '((='
    texenv.comment_end_string = '=))'
    texenv.filters['escape_tex'] = escape_tex


Once setup, you can use the new environment to render your LaTeX
templates.


::

    template = texenv.get_template('template.tex')
    template.render(name='Tom')


Your LaTeX template might look like this:


::

    \documentclass{article}
    
    \begin{document}
    Hello, ((( name|escape_tex )))!
    \end{document}


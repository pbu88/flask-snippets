Enable jinja2 line statements
=============================

Add this line of code right after creating your app:


::

    app.jinja_env.line_statement_prefix = '%'


Then you can use a single `%` rather than `{% %}` in templates:


::

    <ul>
      % for item in items
        <li>{{ item }}</li>
      % endfor
    </ul>


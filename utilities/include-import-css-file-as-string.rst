Include / Import css file as string
===================================

Here a solution how to include your css file(s) as a string into your
jinja2 template:

i've got this from mitsuhiko. thx to him.

in your .py file define this function:


::

    app = Flask(__name__)
    
    def get_resource_as_string(name, charset='utf-8'):
        with app.open_resource(name) as f:
            return f.read().decode(charset)
    
    app.jinja_env.globals['get_resource_as_string'] = get_resource_as_string


in your template include the file with this:


::

    <style type=text/css>{{ get_resource_as_string('static/styles.css') }}</style>


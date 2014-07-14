Custom HTTP methods
===================

When you design the API for your website, it often makes sense to use
things like the DELETE method to delete resources. Unfortunately, the
HTML specification only allows GET and POST for form methods.

With this snippet, you can write your forms like this:


::

    <form action="{{ url_for('delete_entry', id=10) }}" method="POST">
        <input type="hidden" name="_method" value="DELETE" />
        <input type="submit" value="Delete entry 10" />
    </form>


And use it in your view like this:


::

    @app.route('/entries/<int:id>', methods=['GET'])
    def get_entry(id):
        ...
    
    @app.route('/entries/<int:id>', methods=['POST'])
    def update_entry(id):
        ...
    
    @app.route('/entries/<int:id>', methods=['DELETE'])
    def delete_entry(id):
        ...


The magic snippet to get this working is this:


::

    @app.before_request
    def before_request():
        method = request.form.get('_method', '').upper()
        if method:
            request.environ['REQUEST_METHOD'] = method
            ctx = flask._request_ctx_stack.top
            ctx.url_adapter.default_method = method
            assert request.method == method


Please note that this uses non-public parts of Flask's API, and hence
could easily break in future versions (tested against development
version of 0.2).

(suggestions for improvement welcome!)
.. _http://flask.pocoo.org/snippets/38/: http://flask.pocoo.org/snippets/38/


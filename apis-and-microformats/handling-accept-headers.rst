Handling Accept Headers
=======================

If you send an HTTP request to this website it will send you some HTML back which your browser will render. However if you provide an Accept header and give application/json a higher quality than HTML or any other mimetype (or specify it as only accepted mimetype) it will send you back some JSON instead.

The Helper Function
How does this magic work? With this little helper function:

.. code-block:: python

    from flask import request
    
    def request_wants_json():
        best = request.accept_mimetypes \
            .best_match(['application/json', 'text/html'])
        return best == 'application/json' and \
            request.accept_mimetypes[best] > \
            request.accept_mimetypes['text/html']

Why check if json has a higher quality than HTML and not just go with the best match? Because some browsers accept on */* and we don't want to deliver JSON to an ordinary browser.

Usage
You can easily use this in your functions like this:

.. code-block:: python

    from flask import jsonify, render_template
    
    @app.route('/')
    def show_items():
        items = get_items_from_database()
        if request_wants_json():
            return jsonify(items=[x.to_json() for x in items])
        return render_template('show_items.html', items=items)

I recommend implementing to_json methods on your objects that return a Python object (dictionary etc.) that can be safely converted to JSON by jsonify.

Testing
To test if that works you can use the curl command line utility:

.. code-block:: bash

    $ curl -H accept:application/json http://localhost:5000/

Support PyMongo ObjectIDs in URLs
=================================

If you are using MongoDB with Flask you might want to consider adding
native support for ObjectIDs to the routing system. With that object
IDs will be compressed in URLs and the routing system will reject
invalid ObjectIDs automatically.


Converter Implementation
------------------------


::

    from flask import Flask
    from werkzeug.routing import BaseConverter, ValidationError
    from itsdangerous import base64_encode, base64_decode
    from bson.objectid import ObjectId
    from bson.errors import InvalidId
    
    class ObjectIDConverter(BaseConverter):
        def to_python(self, value):
            try:
                return ObjectId(base64_decode(value))
            except (InvalidId, ValueError, TypeError):
                raise ValidationError()
        def to_url(self, value):
            return base64_encode(value.binary)




Example Usage
-------------

And here is a minimal example that shows how it works:


::

    from flask import Flask
    
    app = Flask(__name__)
    app.url_map.converters['objectid'] = ObjectIDConverter
    
    @app.route('/users/<objectid:user_id>')
    def show_user(user_id):
        return 'User ID: %r' % user_id


The object IDs are then automatically compressed by `url_for` and
decompressed by the routing system. To test this you can navigate to
`http://localhost:5000/users/UQWFb53NMX6iFe0s`_ in the above example
which will then expand to `ObjectId('5105856f9dcd317ea215ed2c')`
.. _http://localhost:5000/users/UQWFb53NMX6iFe0s: http://localhost:5000/users/UQWFb53NMX6iFe0s


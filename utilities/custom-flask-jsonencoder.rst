Custom Flask JSONEncoder
========================

As the default Flask json encoder serialized datetime as RFC 822
datetime strings, Sometimes we need serialized datetime in other
format. The snippet below is show how to custom Flask json encoder.


Defualt JSONEncoder
~~~~~~~~~~~~~~~~~~~


::

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from flask import Flask, jsonify
    from datetime import datetime
    
    app = Flask(__name__)
    
    
    @app.route('/default')
    def default_jsonencoder():
        now = datetime.now()
        return jsonify({'now': now})
    
    
    if __name__ == '__main__':
        app.run(debug=True)


Here is the default result:

::

    http://127.0.0.1:5000/default
    
    {
      "now": "Thu, 28 Nov 2013 22:28:43 GMT"
    }



Custom JSONEncoder
~~~~~~~~~~~~~~~~~~


::

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
    from flask import Flask, jsonify
    from flask.json import JSONEncoder
    import calendar
    from datetime import datetime
    
    
    class CustomJSONEncoder(JSONEncoder):
    
        def default(self, obj):
            try:
                if isinstance(obj, datetime):
                    if obj.utcoffset() is not None:
                        obj = obj - obj.utcoffset()
                    millis = int(
                        calendar.timegm(obj.timetuple()) * 1000 +
                        obj.microsecond / 1000
                    )
                    return millis
                iterable = iter(obj)
            except TypeError:
                pass
            else:
                return list(iterable)
            return JSONEncoder.default(self, obj)
    
    app = Flask(__name__)
    app.json_encoder = CustomJSONEncoder
    
    
    @app.route('/custom')
    def custom_jsonencoder():
        now = datetime.now()
        return jsonify({'now': now})
    
    if __name__ == '__main__':
        app.run(debug=True)


Here is the custom result:

::

    http://127.0.0.1:5000/custom
    
    {
      "now": 1385677404333
    }


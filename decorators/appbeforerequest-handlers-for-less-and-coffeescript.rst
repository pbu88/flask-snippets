app.before_request handlers for less and coffeescript
=====================================================

Hi,

I am using less and coffee in one of my projects. I found Steve Losh's
extension which does the less compilation `https://github.com/sjl
/flask-lesscss/blob/master/flaskext/lesscss.py`_ and I read somewhere
there is another based on it for coffeescript.

The extension, though quite simple, does a os.walk for each request
irrespective of what the request does.

I am using a slightly modified version which doesn't do an os.walk and
handles blueprints.

EDIT: I can't figure out how to paste code.

`https://gist.github.com/2730487`_
.. _https://gist.github.com/2730487: https://gist.github.com/2730487
.. _https://github.com/sjl/flask-lesscss/blob/master/flaskext/lesscss.py: https://github.com/sjl/flask-lesscss/blob/master/flaskext/lesscss.py


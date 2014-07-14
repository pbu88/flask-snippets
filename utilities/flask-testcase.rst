Flask TestCase
==============

This is a subclass of unittest.TestCase to help manage your unit tests
in Flask projects.

It sets up the application and client and exposes the request context
so you can do app-specific things outside of your views.


::

    import unittest
    
    class TestCase(unittest.TestCase):
    
        def create_app(self):
            """
            Create your Flask app here, with any
            configuration you need
            """
            raise NotImplementedError
    
        def __call__(self, result=None):
           """
           Does the required setup, doing it here
           means you don't have to call super.setUp
           in subclasses.
           """
           self._pre_setup()
           super(TestCase, self).__call__(result)
           self._post_tearDown()
    
        def _pre_setup(self):
           self.app = self.create_app()
           self.client = self.app.test_client()
           
           # now you can use flask thread locals
    
           self._ctx = self.app.test_request_context()
           self._ctx.push()
    
        def _post_tearDown(self):
           self._ctx.pop()


A further step would be to add convenience methods to this TestCase -
for example assertRedirects or assert404 :


::

        def assert404(self, response):
            """
            Checks if a HTTP 404 returned
            e.g. 
            resp = self.client.get("/")
            self.assert404(resp)
            """
            self.assertTrue(response.status_code == 404)


If you need to handle fixtures with SQLAlchemy or another ORM/backend
then the Fixture package may be of use:

`http://pypi.python.org/pypi/fixture/1.3.1`_
.. _http://pypi.python.org/pypi/fixture/1.3.1: http://pypi.python.org/pypi/fixture/1.3.1


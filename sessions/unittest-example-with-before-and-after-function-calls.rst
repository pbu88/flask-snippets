Unittest example with before and after function calls
=====================================================

The following has been tested with Flask 6 but I think that it should
work with other releases.

::

    def test_set_date_range(self):
        arg_dict = {
                'min_date': "2011-7-1",
                'max_date': "2011-7-4",
        }
        with self.app.test_request_context('/date_range/',
                    method="POST", data=arg_dict):
    
            # call the before funcs
            rv = self.app.preprocess_request()
            if rv != None:
                response = self.app.make_response(rv)
            else:
                # do the main dispatch
                rv = self.app.dispatch_request()
                response = self.app.make_response(rv)
    
                # now do the after funcs
                response = self.app.process_response(response)
    
        assert response.mimetype == 'application/json'
        assert "OK" in response.data


*There should be a "Testing" category."*


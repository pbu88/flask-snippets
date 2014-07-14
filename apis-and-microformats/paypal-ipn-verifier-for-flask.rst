Paypal IPN Verifier for Flask
=============================

I had been trying to verify paypal IPN using Flask. After researching
a bit, i found a good `snippet`_ in GIST, by `cbsmith`_. I did a
little modification to make it work in all cases.


::

    IPN_URLSTRING = 'https://www.sandbox.paypal.com/cgi-bin/webscr'
    IPN_VERIFY_EXTRA_PARAMS = (('cmd', '_notify-validate'),)
    from itertools import chain
    
    def ordered_storage(f):
        import werkzeug.datastructures
        import flask
        def decorator(*args, **kwargs):
            flask.request.parameter_storage_class = werkzeug.datastructures.ImmutableOrderedMultiDict
            return f(*args, **kwargs)
        return decorator
    
    @app.route('/paypal/', methods=['POST'])
    @ordered_storage
    def paypal_webhook():
        #probably should have a sanity check here on the size of the form data to guard against DoS attacks
        verify_args = chain(request.form.iteritems(), IPN_VERIFY_EXTRA_PARAMS)
        verify_string = '&'.join(('%s=%s' % (param, value) for param, value in verify_args))
        #req = Request(verify_string)
        response = urlopen(IPN_URLSTRING, data=verify_string)
        status = response.read()
        print status
        if status == 'VERIFIED':
            print "PayPal transaction was verified successfully."
            # Do something with the verified transaction details.
            payer_email =  request.form.get('payer_email')
            print "Pulled {email} from transaction".format(email=payer_email)
        else:
             print 'Paypal IPN string {arg} did not validate'.format(arg=verify_string)
    
        return jsonify({'status':'complete'})
.. _cbsmith: https://gist.github.com/cbsmith
.. _snippet: https://gist.github.com/cbsmith/5069769


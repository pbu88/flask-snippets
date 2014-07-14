Flask Set Cookies by Response
=============================


::

    @app.route('/set_cookie')
    def cookie_insertion():
        redirect_to_index = redirect('/index')
        response = current_app.make_response(redirect_to_index )  
        response.set_cookie('cookie_name',value='values')
        return response


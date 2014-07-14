Flashing errors from WTForms forms
==================================

The `flash` function is really useful, as it makes it easy to deliver
notifications in a nice, easy way, and guarantee that the user will
see it. Normally, when you use WTForms, you have to display errors
from your template by showing them in the template next to the field
or at the top of the form. But if you want to use the flash mechanism
to show the errors, just call this on the form from within the view:


::

    def flash_errors(form):
        for field, errors in form.errors.items():
            for error in errors:
                flash(u"Error in the %s field - %s" % (
                    getattr(form, field).label.text,
                    error
                ))


That way, your site can have a nice, uniform user interface for
notifications.


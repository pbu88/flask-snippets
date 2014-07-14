Generating PDF from Flask template (using xhtml2pdf)
====================================================

*Please note:*

*There are `many ways`_ for generating PDF in python. This snippet
aims at presenting one of the simplest: with xhtml2pdf.*

*`xhtml2pdf`_ is a fork from an earlier project (called "pisa"). The
project packaging is a bit confusing and it's not very well
documented, but it allows you to generate nice PDFs, simply, without
having to learn all the `ReportLab`_ doc. The small documentation
(inherited from pisa) can be found `here`_.*


How to generate a PDF file from a Flask template using xhtml2pdf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First, create a simple Jinja template at `your/template.html`. For
instance it may contain:


::

    <html>
        <head>
        </head>
        <body>
        {% block body %}
            <h2>Some page title.</h2>
            <table width="100%" cellpadding="4" cellspacing="0">
                <tbody>
                    <tr>
                        <td width="100%">Some text.</td>
                    </tr>
                </tbody>
                <tbody>
                    <tr style="border: 1px solid #000000;">
                        <td width="100%">
                            <h3 style="text-align: center;">Some subtitle</h3>
                            <div>{{ g.user.get('user_attribute') }}</div>
                        </td>
                    </tr>
                </tbody>
            </table>
            <hr style="margin: 3em 0;"/>
            <img src="{{ url_for('static', filename='img.png', _external=True) }}" />
        {% endblock %}
        </body>
    </html>


xhtml2pdf also works with a separate CSS file. All the CSS and (X)HTML
tags supported are listed in the `documentation`_.

Then create a `pdfs.py` file containing:


::

    from xhtml2pdf import pisa
    from cStringIO import StringIO
    
    def create_pdf(pdf_data):
        pdf = StringIO()
        pisa.CreatePDF(StringIO(pdf_data.encode('utf-8')), pdf)
        return pdf


Note that you should use Celery on the `create_pdf` function (adding
the `@task` decorator), because the generation process may be a long
operation.

Now you can call `create_pdf` to create the PDF file from your Flask
template (for instance, for an email attachement):


::

    from flask import Flask, render_template, redirect, url_for
    from flaskext.mail import Mail, Message
    from pdfs import create_pdf
    # ...
    
    app = Flask(__name__)
    mail_ext = Mail(app)
    # ...
    
    @app.route('/your/url')
    def your_view():
        subject = "Mail with PDF"
        receiver = "receiver@mail.com"
        mail_to_be_sent = Message(subject=subject, recipients=[receiver])
        mail_to_be_sent.body = "This email contains PDF."
        pdf = create_pdf(render_template('your/template.html'))
        mail_to_be_sent.attach("file.pdf", "application/pdf", pdf.getvalue())
        mail_ext.send(mail_to_be_sent)
        return redirect(url_for('other_view'))


Note that email sending should also be "celeryed" (see `this thread`_
for more information). ;)
.. _this thread: http://flask.pocoo.org/mailinglist/archive/2011/8/6/flask-celery/#ebde0ad46457edde12533eebd1f3ba39
.. _xhtml2pdf: http://www.xhtml2pdf.com/
.. _many ways: http://stackoverflow.com/questions/2252726/how-to-create-pdf-files-in-python
.. _ReportLab: http://www.reportlab.com/software/opensource/rl-toolkit/
.. _documentation: https://github.com/chrisglass/xhtml2pdf/tree/master/doc


CMS Pages
=========

An idea to create pseudo dynamic text for pseudo static html pages
(index, about us, contact, etc.).

Below the sqlalchemy models:

::

    class Page(Base):
    	__tablename__='pages'
    	id Column(Integer, primary_key=True)
    	name = Column(String())
    	page_snippets = relationship('PageSnippets', backref='pages', lazy='dynamic')
    
    class PageSnippets('base')
    	__tablename__='page_snippets'
    	id Column(Integer, primary_key=True)
    	snippet = Column(String())
    	language = Column(String())


The view:

::

    @app.route('/<page_id>')
    def a_page(page_id):
    	page = Page.query.filter_by(id=page_id).one()
    	page_snippets = page.page_snippets
    	return render_template('a_page.html', page_snippets=page_snippets)


In the template:

::

    <p>{{page_snippets.intro}}</b>
    <div class="footer">
    <p>{{page_snippets.footer}}
    </div>


A jinja macro could be used to filter languages.


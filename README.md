What is pagination?
---------------------
This module helps dividing large lists of items into pages. The user is shown one page at a time and
can navigate to other pages. Imagine you are offering a company phonebook and let the user search
the entries. If the search result contains 23 entries but you may want to display no more than 10
entries at once. The first page contains entries 1-10, the second 11-20 and the third 21-23. See the
documentation of the "Page" class for more information. 

How do I use this module?
---------------------------
The paginate module contains extensive in-line documentation with examples.

Concerning WebHelpers
-----------------------
This is a standalone module. Former versions were included in the WebHelpers Python module as
webhelpers.paginate and were tightly coupled with the WebHelpers and the Pylons web framework. This
version aims to be useful independent of any web framework.

Subclassing Page()
------------------
This module supports pagination through list-like objects. To paginate though other types of objects
you can subclass the paginate.Page() class and provide a wrapper class that defines how to access
elements of that special collection.

You can find examples in other paginate_* modules like paginate_sqlalchemy. Basically you would have
to provide a class that implements the __init__, __getitem__ and __len__ methods.

It is trivial to make pagination for other datastores like Elasticsearch/Solr extending the base class.

Example::

    class SqlalchemyOrmWrapper(object):
        """Wrapper class to access elements of a collection."""
        def __init__(self, obj):
            self.obj = obj

        def __getitem__(self, range):
            # Return a range of objects of an sqlalchemy.orm.query.Query object
            return self.obj[range]

        def __len__(self):
            # Count the number of objects in an sqlalchemy.orm.query.Query object
            return self.obj.count()

Then you can create your own Page class that uses the above wrapper class::

    class SqlalchemyOrmPage(paginate.Page):
        """A pagination page that deals with SQLAlchemy ORM objects."""
        def __init__(self, *args, **kwargs):
            super(SqlalchemyOrmPage, self).__init__(*args, wrapper_class=SqlalchemyOrmWrapper, **kwargs)
    
As you can see it does not do much. It basically calls paginate.Page.__init__ and adds
wrapper_class=SqlalchemyOrmWrapper as an argument. The paginate.Page instance will use that wrapper
class to access the elements.


Generating HTML come for current page
-------------------------------------

Example::

    p = paginate.Page([], page=15, items_per_page=15, item_count=1010)
    # item_count is optional, but we pass a dummy empty resultset for this example
    pattern = '$link_first $link_previous ~4~ $link_next $link_last (Page $page our of $page_count - total $item_count)'
    p.pager(pattern, url='http://foo.com?x=$page', dotdot_attr={'x':5}, link_attr={'y':6}, curpage_attr={'z':77})
    # *_attr arguments are optional and can be used to attach additional classes/attrs to tags


Results in::

    '<a class="L" href="URL?x=1">&lt;&lt;</a> <a class="L" href="URL?x=14">&lt;</a> <a class="L" href="URL?x=1">1</a> <span class="D">..</span> <a class="L" href="URL?x=11">11</a> <a class="L" href="URL?x=12">12</a> <a class="L" href="URL?x=13">13</a> <a class="L" href="URL?x=14">14</a> <span class="C">15</span> <a class="L" href="URL?x=16">16</a> <a class="L" href="URL?x=17">17</a> <a class="L" href="URL?x=18">18</a> <a class="L" href="URL?x=19">19</a> <span class="D">..</span> <a class="L" href="URL?x=68">68</a> <a class="L" href="URL?x=16">&gt;</a> <a class="L" href="URL?x=68">&gt;&gt;</a> (Page 15 our of 68 - total items 1010)'

Using url maker to generate links to specific result ranges
-----------------------------------------------------------

You can pass `url_maker` Callback to generate the URL of other pages, given its numbers.
Must accept one int parameter and return a URI string.

Example::

    def url_maker(page_number):
        return str('foo/%s' % page_number)
    page = paginate.Page(range(100), page=1, url_maker=url_maker)
    eq_(page.pager(), '1 <a href="foo/2">2</a> <a href="foo/3">3</a> .. <a href="foo/5">5</a>')



Alternatively if you will not pass the link builder function, the pager() method can also accept `url` argument that contains URL that page links will point to.
Make sure it contains the string $page which will be replaced by the actual page number.
Must be given unless a url_maker is specified to __init__, in which case this parameter is ignored.

Using link information for custom paginator templates
-----------------------------------------------------

If you do not like the default HTML format produced by paginator you can use link_map() function to generate
a dictionary of links you can use in your own template.

Example::

    p.link_map('$link_first $link_previous ~4~ $link_next $link_last (Page $page our of $page_count - total items $item_count)',url='URL?x=$page',dotdot_attr={'class':'D'}, link_attr={'class':"L"}, curpage_attr={'class':"C"})

Returns something like::

    {'current_page': {'attrs': {'class': 'C'}, 'href': 'URL?x=15', 'value': 15},
     'first_page': {'attrs': {'class': 'L'}, 'href': 'URL?x=1', 'type': 'first_page', 'value': 1},
     'last_page': {'attrs': {'class': 'L'}, 'href': 'URL?x=68', 'type': 'last_page', 'value': 68},
     'next_page': {'attrs': {'class': 'L'}, 'href': 'URL?x=16', 'type': 'next_page', 'value': 16},
     'previous_page': {'attrs': {'class': 'L'}, 'href': 'URL?x=14', 'type': 'previous_page', 'value': 14},
     'range_pages': [{'attrs': {'class': 'D'}, 'href': '', 'type': 'span', 'value': '..'},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=11', 'type': 'page', 'value': '11'},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=12', 'type': 'page', 'value': '12'},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=13', 'type': 'page', 'value': '13'},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=14', 'type': 'page', 'value': '14'},
      {'attrs': {'class': 'C'}, 'href': 'URL?x=15', 'type': 'current_page', 'value': 15},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=16', 'type': 'page', 'value': '16'},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=17', 'type': 'page', 'value': '17'},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=18', 'type': 'page', 'value': '18'},
      {'attrs': {'class': 'L'}, 'href': 'URL?x=19', 'type': 'page', 'value': '19'},
      {'attrs': {'class': 'D'}, 'href': '', 'type': 'span', 'value': '..'}]}


Using link_tag callable to generate custom link markup
------------------------------------------------------

In case you want to generate custom link markup for your project - for example for use with bootstrap,
`pager()` accepts `link_tag` argument that expects a callable that can be used to easly override the way links are
generated.


Example::

    from paginate import Page, make_html_tag

    def paginate_link_tag(item):
        """
        Create an A-HREF tag that points to another page usable in paginate.
        """
        a_tag = Page.default_link_tag(item)
        if item['type'] == 'current_page':
            return make_html_tag('li', a_tag, **{'class':'active'})
        return make_html_tag('li', a_tag)

    paginator.pager(
    curpage_attr={'class':'current_page'},
    dotdot_attr={'class':'spacer'},
    symbol_first='<i class="fa fa-chevron-circle-left"></i>',
    symbol_last='<i class="fa fa-chevron-circle-right"></i>',
    symbol_previous='<i class="fa fa-chevron-left"></i>',
    symbol_next='<i class="fa fa-chevron-right"></i>',
    link_tag=paginate_link_tag)


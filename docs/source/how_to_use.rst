How do I use it?
================

Using beekeeper is pretty simple, but because it can work with almost
any RESTful API, it's also a little tricky to describe. Let's take
a hypothetical API, for FooBar Ventures.

FooBar Ventures is in the business of widget manufacturing; their
API provides tools to help their customers know what kinds of widgets
are available, and gives detailed information about them. Each widget
is compatible with a variety of products from other vendors, and FooBar
Ventures also maintains a list of compatible products which can be accessed
via the API.

First, install beekeeper:

.. code:: bash

    $ pip install beekeeper

Then, from within Python, we'll need to import beekeeper and initialize the FooBar Ventures API:

.. code:: python

    >>> from beekeeper import API

        >>> fbv = API.from_domain('foobar.com')

    Note that if FooBar Ventures served their API over HTTP rather than over HTTPS,
    you'd need to set the "require_https" keyword argument to False to prevent beekeeper
    from raising an exception. Because hive files change the behavior of your
    application, secure transmission is really important. If you're possibly
    going to be passing sensitive information with your application, and the API
    provider doesn't host their hive using HTTP, it may be better for you to download
    their hive yourself, inspect it, include it with your application, and then
    initialize with a statement like this:
    
    >>> fbv = API.from_domain('foobar.com')

Note that if FooBar Ventures served their API over HTTP rather than over HTTPS,
you'd need to set the "require_https" keyword argument to False to prevent beekeeper
from raising an exception. Because hive files change the behavior of your
application, secure transmission is really important. If you're possibly
going to be passing sensitive information with your application, and the API
provider doesn't host their hive using HTTP, it may be better for you to download
their hive yourself, inspect it, include it with your application, and then
initialize with a statement like this:

.. code:: python

    >>> fbv = API.from_hive_file(file_location)

You can also host the hive yourself securely, and initialize like this:

.. code:: python

    >>> fbv = API.from_remote_hive('https://mydomain.tld/fbv_hive.json')

During the initialization, if the API you're accessing requires any
variables declared by the hive, you can pass those in as arguments or keyword
arguments, and those values will be used on any future requests. This process
is similar to what happens when executing a request - more on that later.

Then, let's say we want to get a list of all the widgets that FooBar
makes:

.. code:: python

    >>> fbv.Widgets.list()

    ['RT6330', 'PV46', 'GX280']

We didn't need to pass any special variables to the API for this request
outside of what's automatically handled already, so it's very simple.
Beekeeper also handles parsing the returned data into a Pythonic format, so
it's easy to iterate across and subscript into.

Now, I see one widget I think I'm interested in, called the GX280. Before
going further, though, I want to make sure that it's compatible with my
system, a HyperStar HS2000.

.. code:: python

    >>> fbv.Widgets['GX280'].compatiblilty_list()

    {'manufacturers': {'Athena': {'CompatibleModels': ['AM4000', 'AM236', 'AM236b']}, 'HyperStar': {'CompatibleModels': ['HS133', 'HS450', 'HS3200', 'HS2000']}}}

Yikes, that's a big response. I could probably parse through it, but a), I'm
kind of lazy, and b), maybe there's an easier way. You'll note something
interesting about the request first, though; it has a dictionary-style subscription
in the middle. This is because FooBar Ventures was kind enough, when they
wrote their hive file, to define a ID variable for the Widget object.
What this means is that if I know the ID for an object, I can easily get to
that particular instance of an object, just by subscripting.

To deal with the response? I mentioned I'm a bit lazy, so I took a quick
look at the API documentation, and it looks like FooBar provides a method
to direcly check compatibility for a particular model. Let's do that instead:

.. code:: python

    >>> fbv.Widgets['GX280'].compatible_with('HS2000')

    {'compatible': True, 'widgetModel': 'GX280', 'systemModel': 'HS2000'}

That's easier! Now, it looks like my system is compatible with that widget,
so I want to take a closer look at it; make sure it's a good fit. I don't
really care about other widgets at the moment, so I'm going to make it
a bit easier by assigning the API object instance for the GX280 to its
own variable:

.. code:: python

    >>> gx280 = fbv.Widgets['GX280']

Note that this isn't downloading any data; it's just binding all the actions
that are associated with that particular object, and all the variables
that need to be in place for those actions to work, to the name I picked. I
can then use any actions as if I had typed out the whole long thing.

.. code:: python

    >>> gx280.description()

    {'widgetModel': 'GX280', 'description': 'It's super cool!'}

GUYS, IT'S SUPER COOL. I MUST HAVE IT. I think I need 20 of them.

.. code:: python

    >> gx280.order(20)

    TypeError: Expected values for variables: ['cc_number', 'quantity']

Oh. I guess they want to be paid.

Up until now, we've just been dealing with cases where we need to fill in
one variable. When that's the case, beekeeper doesn't even make you tell
it the variable name. But when we have more than one variable, you do need
to fill that in. Let's try again:

.. code:: python

    >>> gx280.order(quantity=20, cc_number=1234234534564567)

    {'status': 'OrderCreated', 'OrderNumber': 5960283}

There we go!

Note that I didn't actually need to fill in the name for "quantity". Because
I filled in the name for "cc_number" (the only other required variable),
beekeeper could have figured out that a variable out on its own without
a name should go to the Quantity field. Or, vice versa. If I had filled in
"quantity=20", beekeeper would have figured out that the other variable
should go into "cc_number".

And that's all there is to using beekeeper! It's simple, fast, and makes
working with remote APIs much, much, much easier.

If you're not sure what objects and actions are available for an API, you can
easily see the structure by just doing the following:

.. code:: python

    >>> print(fbv)

    FooBar Ventures()
    |
    |---Widgets[widget_id]
    |   |   A widget, made by FooBar Ventures!
    |   |
    |   |---list()
    |   |       Get a list of all widgets
    |   |
    |   |---compatibility_list(widget_id)
    |   |       Get a list of systems compatible with the given widget
    |   |
    |   |---compatible_with(widget_id, system_id)
    |   |       Is the system compatible with the widget?
    |   |
    |   |---description(widget_id)
    |   |       Get a description of the widget
    |   |
    |   |---order(widget_id, cc_number, quantity)
    |   |       Order the given quantity of the widget

It'll give you a nice printout so you can see where you need to go, and what
variable values you need to get there.

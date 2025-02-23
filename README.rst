beekeeper |Build Status| |Read Docs|
====================================

Description
-----------

beekeeper is a Python library designed around dynamically generating a
RESTful client interface based on a minimal JSON hive.

The hive specification is designed to provide beekeeper (or other
applications consuming hive files) with programmatically-designed
insight into the structure of both the REST endpoints that are available
and the objects and methods that those endpoints represent.

While the classes available in beekeeper can be used manually to create
Pythonic representations of REST endpoints, it is strongly preferred
that the library be used as a whole with a constructed hive file. As
APIs become larger in scale (in terms of the number of endpoints and
represented objects), the time benefit of beekeeper becomes more
pronounced, as adding additional objects and endpoints is a trivial
process.

Requirements
------------

beekeeper requires Python 2.7.9/3.4.3 or higher and their built-in
modules, as well as xmltodict.

Installation
------------

.. code:: python

   pip install beekeeper

Usage
-----

The usage of beekeeper will depend on what features are provided by the
person who wrote the hive file. There are a number of ways that the hive
writer can make your life easier. Regardless, at a base level, usage will
look something like this:

.. code:: python

    from beekeeper import API
    myAPI = API.from_hive_file('fname.json')
    x = myAPI.Widgets.action(id='foo', argument='bar')

If the hive developer defines an ID variable for the object you're working
with, you can subscript, dictionary style:

.. code:: python

    x = myAPI.Widgets['foo'].action(argument='bar')

If you've only got one remaining argument in the method call, you don't even
need to name it! You can do something like this:

.. code:: python

   x = myAPI.Widgets['foo'].action('bar')

This also holds true if you have multiple variables, but the other ones are
assigned by name:

.. code:: python

   x = myAPI.Widgets['foo'].action('bar', var2='baz')

If you're using a hive file, then it should define which variables are needed.
If you try to call a function without filling in that variable, it should
automatically yell at you and tell you what variables are missing. Since these
variables are defined within the hive, beekeeper will do the work for you, 
automatically determine what data type a particular variable is, and put it
exactly where it needs to go.

beekeeper will also automatically handle parsing data. When you
send data, beekeeper will read the MIME type that was defined in the variable
for that data, and try to automatically move it from a "Python" format (e.g., 
a dictionary) to the right REST API format (e.g., JSON).

This holds true in the other direction as well; beekeeper will read the MIME
type of the response data, and hand it back to you in a Pythonic format! If
beekeeper doesn't know how to handle the data, it'll just give you the raw
bytes so that you can do what you need to with them.

Notes
-----

beekeeper does not currently do SSL certificate verification when used
on Python versions earlier than 2.7.9 or 3.4.3.

.. |Build Status| image:: https://travis-ci.org/haikuginger/beekeeper.svg?branch=master
   :target: https://travis-ci.org/haikuginger/beekeeper

.. |Read Docs| image:: https://readthedocs.org/projects/beekeeper/badge/?version=latest
    :target: http://beekeeper.readthedocs.org/en/latest/?badge=latest
    :alt: Documentation Status
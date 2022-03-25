Data formats
============

In-memory data
--------------

Plottr's data management is entirely based on the DataDict object. The DataDict object is built on top of
python dictionary (MAYBE ADD A LINK TO THE PYTHON DICTIONARY PAGE) so it can do the same things as a normal dictionary
can with some extra features and restrictions designed for scientific use(?) QUESTION MARK HERE.
Inheriting the DataDictBase class as a parent, you can implement your own restriction and helper methods for your own
needs.

DataDict
^^^^^^^^

The DataDict is the most basic form of ``DataDicts``. We can think of it a dictionary containing other dictionaries.
Add example of how it looks like inside from the plottr documentation.

Basic Use
~~~~~~~~~

We can start by creating an empty DataDict like any other python object:

>>> data_dict = DataDict()
>>> data_dict
{}

We can create the structure of the data_dict by creating new dictionary keys and populating them like a normal python
dictionary.

>>> data_dict['x'] = dict(unit='m')
>>> data_dict
{'x': {'unit': 'm'}}

We can also start by creating a ``DataDict`` that has the structure of the data we are going to record but no data yet:

>>> data_dict = DataDict(x=dict(unit='m'), y = dict(unit='m'), z = dict(axes=['x', 'y']))
>>> data_dict
{'x': {'unit': 'm'}, 'y': {'unit': 'm'}, 'z': {'axes': ['x', 'y']}}

The ``DataDict`` that we just created contains no data yet, only the structure of how the data will look like. We have
also specified the unit of ``x`` and ``y`` and which variables are independent variables (``x``, ``y``), or how we will
call them from now on, ``axes`` and dependent variables (``z``), or, ``dependents``.

Structure
~~~~~~~~~

From the basic and empty ``DataDict`` we can already start to inspect its structure. To see the entire structure of a
``DataDict`` we can use the ``structure()`` method:

>>> data_dict = DataDict(x=dict(unit='m'), y = dict(unit='m'), z = dict(axes=['x', 'y']))
>>> data_dict.structure()
{'x': {'unit': 'm', 'axes': [], 'label': ''},
 'y': {'unit': 'm', 'axes': [], 'label': ''},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': ''}}

We can check for specific things inside the ``DataDict``. We can look at the axes:

>>> data_dict.axes()
['x', 'y']

We can look at all the dependents:

>>> data_dict.dependents()
['z']

We can also see the shape of a ``DataDict`` by using the ``shapes()`` method:

>>> data_dict.shapes()
{'x': (0,), 'y': (0,), 'z': (0,)}

Populating the DataDict
~~~~~~~~~~~~~~~~~~~~~~~

One of the only "restrictions" that the basic ``DataDict`` implements is that every data field must have the same number
of records (items). However, restrictions is in quotes because there is nothing that is actually stopping you from
having different data fields have different number of records, this will only make the ``DataDict`` invalid.
We will explore what his means later.

There are 2 different ways of populating a ``DataDict``, adding data to it or appending 2 different ``DataDict`` to each
other.

The correct way, or should we say fool proof, of adding data to an existing ``DataDict`` is with the ``add_data()``
method:

>>> data_dict = DataDict(x=dict(unit='m'), y = dict(unit='m'), z = dict(axes=['x', 'y']))
>>> data_dict.add_data(x=[0,1,2], y=[0,1,2], z=[0,1,4])
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'z': {'axes': ['x', 'y'],  'unit': '',  'label': '',  'values': array([0, 1, 4])}}

We can see that we now have a populated ``DataDict``. It is important to notice that this method will also add any of
the missing items that a data field doesn't have (``values``, ``axes``, ``unit``, and ``label``). Populating the
``DataDict`` with this method will also ensure that every item has the number of records and the correct shape, either
by adding ``nan`` to the other data fields or by nesting the data arrays so that the outer most dimension of every
data field has the same number of records.

We can see this in action if we add a single data field with items but no the rest:

>>> data_dict.add_data(x=[9])
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2, 9])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([ 0.,  1.,  2., nan])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([ 0.,  1.,  4., nan])}}

As we can see, both ``y`` and ``z`` have an extra ``nan`` record in them. We can observe the change of dimension if we
do not add the same number of items to all data fields

>>> data_dict = DataDict(x=dict(unit='m'), y = dict(unit='m'), z = dict(axes=['x', 'y']))
>>> data_dict.add_data(x=[0,1,2], y=[0,1,2],z=[0])
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([[0, 1, 2]])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([[0, 1, 2]])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([0])}}

If we check the shapes of the data fields now we can see that both ``x`` and ``y`` have had an extra dimension so that
all data fields contain the same number of records:

>>> data_dict.shapes()
{'x': (1, 3), 'y': (1, 3), 'z': (1,)}


Data storage
------------


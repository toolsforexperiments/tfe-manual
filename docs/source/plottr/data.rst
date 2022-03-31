Data formats
============

In-memory data
--------------

Plottr's data management is entirely based on the DataDict object. The DataDict object is built on top of
python dictionary so it can do the same things as a normal dictionary
can with some extra features and restrictions designed for scientific use(?).
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

If we want to expand our ``DataDict`` by appending another one, we need to make sure that both of our ``DataDicts``
have the same inner structure. We can check that by utilizing the static method ``same_structure()``:

>>> data_dict_1 = DataDict(x=dict(unit='m'), y=dict(unit='m'), z=dict(axes=['x','y']))
>>> data_dict_2 = DataDict(x=dict(unit='m'), y=dict(unit='m'), z=dict(axes=['x','y']))
>>> data_dict_1.add_data(x=[0,1,2], y=[0,1,2], z=[0,1,4])
>>> data_dict_2.add_data(x=[3,4], y=[3,4], z=[9,16])
>>> DataDict.same_structure(data_dict_1, data_dict_2)
True

.. note::
    Make sure that both ``DataDict`` have the exact same structure. This means that every item of every data field that
    appears when using the method ``structure()`` (unit, axes, and label) are identical to one another.
    Any slight difference will make this method fail due to conflicting structures.

The ``append()`` method will do this check before appending the 2 ``DataDict``, and will only append them if the check
returns ``True``. Once we know that the structure is the same we can append them:

>>> data_dict_1.append(data_dict_2)
>>> data_dict_1
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2, 3, 4])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2, 3, 4])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([ 0,  1,  4,  9, 16])}}

Meta Data
~~~~~~~~~

One of the advantages ``DataDicts`` have over regular python dictionaries is their ability to contain meta data.
Meta data can be added to the entire ``DataDict`` or to individual data fields. Any object inside a ``DataDict`` whose
key starts and ends with 2 underscores is considered meta data.

We can simply add meta data manually by adding an item with the proper notation:

>>> data_dict['__metadata__'] = 'important meta data'

Or we can use the ``add_meta()`` method:

>>> data_dict.add_meta('sample_temperature', '10mK')
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([0, 1, 4])},
 '__metadata__': 'important meta data',
 '__sample_temperature__': '10mK'}

We can also add meta data to a specific data field by passing its name as the last argument:

>>> data_dict.add_meta('extra_metadata', 'important meta data', 'x')
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2]), '__extra_metadata__': 'important meta data'},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([0, 1, 4])},
 '__metadata__': 'important meta data',
 '__sample_temperature__': '10mK'}

We can check if a certain meta field exists with the method ``has_meta()``:

>>> data_dict.has_meta('sample_temperature')
True

We can retrieve the meta data with the ``meta_val()`` method:

>>> data_dict.meta_val('sample_temperature')
'10mK'

We can also ask for a meta value from a specific data field by passing the data field in the ``data`` argument:

>>> data_dict.meta_val('extra_metadata','x')
'important meta data'

We can delete a specific meta field by using the ``delete_meta()`` method:

>>> data_dict.delete_meta('metadata')
>>> data_dict.has_meta('metadata')
False

This also work for meta data in data fields by passing the data field as the last argument

>>> data_dict.delete_meta('extra_metadata', 'x')
>>> data_dict['x']
{'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])}

We can delete all the meta data present in the ``DataDict`` with the ``clear_meta()`` method:

>>> data_dict.add_meta('metadata', 'important meta data')
>>> data_dict.add_meta('extra_metadata', 'important meta data', 'x')
>>> data_dict.clear_meta()
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([0, 1, 4])}}

.. note::
    There are 3 helper functions in the datadict module that help converting from meta data name to key.
    These are: ``is_meta_key()``, ``meta_key_to_name()``, and ``meta_name_to_key()``. They are explained in the helper
    functions section.
    TODO: add references to that section of the file once they exist.

Reference
^^^^^^^^^

The reference is split into 2 object since ``DataDict`` is an implementation of ``DataDictBase``. If you are not sure
on how this works see `inheriting <https://docs.python.org/3/tutorial/classes.html#inheritance>`__.

DataDictBase
~~~~~~~~~~~~

The following is the base class from which both ``DataDict`` and ``MeshgridDataDict`` are inheriting.

.. autoclass:: plottr.data.datadict.DataDictBase
    :members:
    :exclude-members: set_meta

DataDict
~~~~~~~~

.. autoclass:: plottr.data.datadict.DataDict
    :members:


Meshgrid DataDict
^^^^^^^^^^^^^^^^^

Extra Module Functions
^^^^^^^^^^^^^^^^^^^^^^



Data storage
------------


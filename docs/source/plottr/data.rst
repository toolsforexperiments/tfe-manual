Data formats
============

In-memory data
--------------

Basic Concept
^^^^^^^^^^^^^

The main format we're using within plottr is the ``DataDict``. While most of the actual numeric data will typically live in numpy arrays (or lists, or similar), they don't typically capture easily arbitrary metadata and relationships between arrays. Say, for example, we have some data ``z`` that depends on two other variables, ``x`` and ``y``. This information has be stored somewhere, and numpy doesn't offer readily a solution here. There are various extensions, for example `xarray <http://xarray.pydata.org>`_ or the `MetaArray class <https://scipy-cookbook.readthedocs.io/items/MetaArray.html>`_. Those however typically have a grid format in mind, which we do not want to impose. Instead, we use a wrapper around the python dictionary that contains all the required meta information to infer the relevant relationships, and that uses numpy arrays internally to store the numeric data. Additionally we can story any other arbitrary meta data.

A DataDict container (a `dataset`) can contain multiple `data fields` (or variables), that have values and can contain their own meta information. Importantly, we distinct between independent fields (the `axes`) and dependent fields (the `data`).

Despite the naming, `axes` is not meant to imply that the `data` have to have a certain shape (but the degree to which this is true depends on the class used). A list of classes for different shapes of data can be found below.

The basic structure of data conceptually looks like this (we inherit from `dict`) ::

        {
            'data_1' : {
                'axes' : ['ax1', 'ax2'],
                'unit' : 'some unit',
                'values' : [ ... ],
                '__meta__' : 'This is very important data',
                ...
            },
            'ax1' : {
                'axes' : [],
                'unit' : 'some other unit',
                'values' : [ ... ],
                ...,
            },
            'ax2' : {
                'axes' : [],
                'unit' : 'a third unit',
                'values' : [ ... ],
                ...,
            },
            '__globalmeta__' : 'some information about this data set',
            '__moremeta__' : 1234,
            ...
        }

In this case we have one dependent variable, ``data_1``, that depends on two axes, ``ax1`` and ``ax2``. This concept is restricted only in the following way:

    * a dependent can depend on any number of independents
    * an independent cannot depend on other fields itself
    * any field that does not depend on another, is treated as an axis

Note that meta information is contained in entries whose keys start and end with double underscores. Both the DataDict itself, as well as each field can contain meta information.

In the most basic implementation, the only restriction on the data values is that they need to be contained in a sequence (typically as list, or numpy array), and that the length of all values in the data set (the number of `records`) must be equal. Note that this does not preclude nested sequences!

Relevant data classes
~~~~~~~~~~~~~~~~~~~~~

:DataDictBase: The main base class. Only checks for correct dependencies. Any
               requirements on data structure is left to the inheriting classes. The class contains methods for easy access to data and metadata.
:DataDict: The only requirement for valid data is that the number of records is the
           same for all data fields. Contains some tools for expansion of data.
:MeshgridDataDict: For data that lives on a grid (not necessarily regular).

DataDict
^^^^^^^^

The DataDict is the most basic implementation of a DataDicts . We can think of it a dictionary containing other dictionaries.

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
``DataDict`` we can use the :meth:`structure <plottr.data.datadict.DataDictBase.structure>` method:

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

We can also see the shape of a ``DataDict`` by using the :meth:`shapes <plottr.data.datadict.DataDictBase.shapes method:

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

The correct way, or should we say fool proof, of adding data to an existing ``DataDict`` is with the :meth:`add_data <plottr.data.datadict.DataDictBase.add_data>` method:

>>> data_dict = DataDict(x=dict(unit='m'), y = dict(unit='m'), z = dict(axes=['x', 'y']))
>>> data_dict.add_data(x=[0,1,2], y=[0,1,2], z=[0,1,4])
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'z': {'axes': ['x', 'y'],  'unit': '',  'label': '',  'values': array([0, 1, 4])}}

We can see that we now have a populated ``DataDict``. It is important to notice that this method will also add any of
the missing items that a data field doesn't have (``values``, ``axes``, ``unit``, and ``label``). Populating the
``DataDict`` with this method will also ensure that every item has the number of records and the correct shape, either
by adding ``NaN`` to the other data fields or by nesting the data arrays so that the outer most dimension of every
data field has the same number of records.

We can see this in action if we add a single data field with items but no the rest:

>>> data_dict.add_data(x=[9])
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2, 9])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([ 0.,  1.,  2., nan])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([ 0.,  1.,  4., nan])}}

As we can see, both ``y`` and ``z`` have an extra ``NaN`` record in them. We can observe the change of dimension if we
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
have the same inner structure. We can check that by utilizing the static method :meth:`same_structure <plottr.data.datadict.DataDictBase.same_structure>`:

>>> data_dict_1 = DataDict(x=dict(unit='m'), y=dict(unit='m'), z=dict(axes=['x','y']))
>>> data_dict_2 = DataDict(x=dict(unit='m'), y=dict(unit='m'), z=dict(axes=['x','y']))
>>> data_dict_1.add_data(x=[0,1,2], y=[0,1,2], z=[0,1,4])
>>> data_dict_2.add_data(x=[3,4], y=[3,4], z=[9,16])
>>> DataDict.same_structure(data_dict_1, data_dict_2)
True

.. note::
    Make sure that both ``DataDict`` have the exact same structure. This means that every item of every data field that
    appears when using the method :meth:`same_structure <plottr.data.datadict.DataDictBase.same_structure>` (unit, axes, and label) are identical to one another.
    Any slight difference will make this method fail due to conflicting structures.

The :meth:`append <plottr.data.DataDict.append>` method will do this check before appending the 2 ``DataDict``, and will only append them if the check
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

Or we can use the :meth:`add_meta <plottr.data.datadict.DataDictBase.add_meta` method:

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

We can check if a certain meta field exists with the method :meth:`has_meta <plottr.data.datadict.DataDictBase.has_meta>`:

>>> data_dict.has_meta('sample_temperature')
True

We can retrieve the meta data with the :meth:`meta_val <plottr.data.datadict.DataDictBase.meta_val>` method:

>>> data_dict.meta_val('sample_temperature')
'10mK'

We can also ask for a meta value from a specific data field by passing the data field in the ``data`` argument:

>>> data_dict.meta_val('extra_metadata','x')
'important meta data'

We can delete a specific meta field by using the :meth:`delete_meta <plottr.data.datadict.DataDictBase.delete_meta>` method:

>>> data_dict.delete_meta('metadata')
>>> data_dict.has_meta('metadata')
False

This also work for meta data in data fields by passing the data field as the last argument

>>> data_dict.delete_meta('extra_metadata', 'x')
>>> data_dict['x']
{'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])}

We can delete all the meta data present in the ``DataDict`` with the :meth:`clear_meta <plottr.data.datadict.DataDictBase.clear_meta>` method:

>>> data_dict.add_meta('metadata', 'important meta data')
>>> data_dict.add_meta('extra_metadata', 'important meta data', 'x')
>>> data_dict.clear_meta()
>>> data_dict
{'x': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'y': {'unit': 'm', 'axes': [], 'label': '', 'values': array([0, 1, 2])},
 'z': {'axes': ['x', 'y'], 'unit': '', 'label': '', 'values': array([0, 1, 4])}}

.. note::
    There are 3 helper functions in the datadict module that help converting from meta data name to key.
    These are: :func:`is_meta_key <plottr.data.datadict.is_meta_key>`, :func:`meta_key_to_name <plottr.data.datadict.meta_key_to_name>` , and :func:`meta_name_to_key <plottr.data.datadict.meta_name_to_key>`.



Meshgrid DataDict
^^^^^^^^^^^^^^^^^

The ``MeshgridDataDict`` is the second implementation of ``DataDictBase`` in the module,
which supports multi-dimensional data in which the data is shaped in a grid where every point of the grid needs to be specified.(see `NumPy method <https://numpy.org/doc/stable/reference/generated/numpy.meshgrid.html>`__)

The implementation is relatively simple to use, it only adds 3 new methods to ``DataDictBase``, ``shape()`` (ADD REF),
``validate()`` and ``reorder_axes()``. Because the tools to populate ``DataDict`` are implementations of it and are
not present in ``DataDictBase``, ``MeshgridDataDict`` lacks the methods ``add_data()`` and ``append()``,
meaning that you will need to populate ``MeshgridDataDict`` like a normal dictionary while still maintaining the basic
``DataDict`` structure of data (ADD REFERENCE HERE TO THE EXAMPLE OF THE STRUCTURE ON THE TOP).

Old docstring:
A dataset where the axes form a grid on which the dependent values reside.

This is a more special case than ``DataDict``, but a very common scenario.
To support flexible grids, this class requires that all axes specify values
for each datapoint, rather than a single row/column/dimension.

For example, if we want to specify a 3-dimensional grid with axes x, y, z,
the values of x, y, z all need to be 3-dimensional arrays; the same goes
for all dependents that live on that grid.
Then, say, x[i,j,k] is the x-coordinate of point i,j,k of the grid.

This implies that a ``MeshgridDataDict`` can only have a single shape,
i.e., all data values share the exact same nesting structure.

For grids where the axes do not depend on each other, the correct values for
the axes can be obtained from `np.meshgrid  <https://numpy.org/doc/stable/reference/generated/numpy.meshgrid.html>`__ (hence the name of the class).

Example: a simple uniform 3x2 grid might look like this; x and y are the
coordinates of the grid, and z is a function of the two::

    x = [[0, 0],
         [1, 1],
         [2, 2]]

    y = [[0, 1],
         [0, 1],
         [0, 1]]

    z = x * y =
        [[0, 0],
         [0, 1],
         [0, 2]]

.. note::
    Internally we will typically assume that the nested axes are
    ordered from slow to fast, i.e., dimension 1 is the most outer axis, and
    dimension N of an N-dimensional array the most inner (i.e., the fastest
    changing one). This guarantees, for example, that the default implementation
    of np.reshape has the expected outcome. If, for some reason, the specified
    axes are not in that order (e.g., we might have ``z`` with
    ``axes = ['x', 'y']``, but ``x`` is the fast axis in the data).
    In such a case, the guideline is that at creation of the meshgrid, the data
    should be transposed such that it conforms correctly to the order as given
    in the ``axis = [...]`` specification of the data.
    The function ``datadict_to_meshgrid`` provides options for that.

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
~~~~~~~~~~~~~~~~~

.. autoclass:: plottr.data.datadict.MeshgridDataDict
    :members:

Extra Module Functions
~~~~~~~~~~~~~~~~~~~~~~

.. automodule:: plottr.data.datadict
    :members:
    :exclude-members: plottr.data.datadict.docstring, DataDictBase, DataDict, MeshgridDataDict, GriddingError


Data storage
------------


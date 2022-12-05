Sweeping
========
.. _labcore sweeping:

Introduction
------------

THIS IS TEMPORARY [MAYBE]

The main task for running experiences utilizing `labcore` is to define what your :class:`Sweep <labcore.measurement.Sweep>` is.
A Sweep object can be iterated over, this is how we execute measurements.
Each iteration performs some actions and return some data in the form of a dictionary, we call this dictionary generated after each step a **record**.

A sweep is composed of 2 parts, a **pointer**, and **actions**:

    * A **pointer** is again an iterable, and represents what our "independent" variable, this represents the variable we control and change to the values specified in the iterator.
    * An **action** is a callable that may take the values the pointer is returning at each iteration as arguments.
      Each action is executed once for each pointer, and you can have as many actions for a single pointer as needed.

Defining and executing a basic sweep would look something like this:

>>> sweep_object = Sweep(range(5), func_1, func_2)
>>> for data in sweep_object:
>>>     print(data)
>>> {variable_1: some_data, variable_2: more_data}
>>> {variable_1: different_data, variable_2: more_different_data}

Where the `range` function is your **pointer** and the two functions are your **actions**.

Once the sweep is created but before execution we can easily infer what records the sweep produces.
Sweeps can be combined in ways that result in nesting, zipping, or appending. Combining sweeps again results in a sweep.
This will allow us to construct modular measurements from pre-defined blocks.
For more information please see :ref:`Introduction to sweeping <labcore sweeping>`

Basic Example
^^^^^^^^^^^^^

A Sweep is created out of two main components, an iterable **pointer** and a variable number of **actions**.
Both **pointer** and **actions** may generate **records**.

The most bare example would look like this:

>>> for data in Sweep(range(3)):
>>>     print(data)
{}
{}
{}

In this example, the range(3) iterable object is our **pointer**.
This Sweep does not contain any **actions** or generate any **records**, but instead simply loops over the iterable **pointer**.

Recording Data
--------------

Concepts
^^^^^^^^

Even though the **pointer** in the previous example does generate data, we cannot see it when we iterate through the sweep.
To have **pointers** and **actions** generate data, we need to indicate to the sweep that they generate **records**.
Each **record** corresponds to a variable, and because of this, we need to tell the Sweep what that **record's** label and its relationship with other variables (whether it's and independent variable or what its dependencies are).

For a Sweep to know that either a **pointer** (an iterable object), or an **action** (a callable object) they need to be wrapped by an instance of :class:`DataSpec <labcore.measurement.record.DataSpec>`.
:class:`DataSpec <labcore.measurement.record.DataSpec>` is a `data class <https://docs.python.org/3/library/dataclasses.html>`__ that holds information about the variable itself.
Understanding the inner workings are not necessary to fully utilize Sweeps, however it is good to know they exists and what information they hold.
The important fields of a :class:`DataSpec <labcore.measurement.record.DataSpec>` are its :class:`name <labcore.measurement.record.DataSpec.name>` and :class:`depends_on <labcore.measurement.record.DataSpec.depends_on>` fields.
:class:`name <labcore.measurement.record.DataSpec.name>`, simply indicates the name of the variable, e.i. the key that the sweep will have for the value of this variable.
:class:`depends_on <labcore.measurement.record.DataSpec.depends_on>` indicates whether the variable is an independent variable (we control) or a dependent variable (the things we are trying to measure).
If `depends_on=None` it means this variable is an independent variable.
If `depends_on=['x']`, this variable is dependent on a separate variable with name `x`.
If `depends_on=[]`, the variable will be automatically assigned as a dependent of all other independents in the same Sweep.

.. note::
    :class:`DataSpec <labcore.measurement.record.DataSpec>`, also contains two more fields: :class:`unit <labcore.measurement.record.DataSpec.unit>` and :class:`type <labcore.measurement.record.DataSpec.type>`, these however have no impact in the way the code behaves and are for adding extra metadata for the user.

While this might seem like a lot of information, its use is very intuitive and easy to use.

Implementation
^^^^^^^^^^^^^^

To wrap functions we use the :class:`recording <labcore.measurement.record.recording>` decorator on the function we want to annotate:

>>> @recording(DataSpec('x'), DataSpec('y', depends_on=['x'], type='array'))
>>> def measure_stuff(n, *args, **kwargs):
>>>     return n, np.random.normal(size=n)
>>>
>>> measure_stuff(1)
{'x': 1, 'y': array([0.70663348])}

In the example above we annotate the function `measure_stuff()` indicating that the first item it returns is `x`, an independent variable since it does not have a `depends_on` field, and the second item is `y`, a variable that depends on `x`.

We can annotate generators in the same way:

>>> @recording(DataSpec('a'))
>>> def make_sequence(n):
>>>     for i in range(n):
>>>         yield i
>>>
>>> for data in make_sequence(3):
>>>     print(data)
{'a': 0}
{'a': 1}
{'a': 2}

A nicer way of creating :class:`DataSpec <labcore.measurement.record.DataSpec>` instances is to use the functions :func:`independent <labcore.measurement.record.independent>` and :func:`dependent <labcore.measurement.record.dependent>`.
This function just makes the recording of data easier to read.
:func:`independent <labcore.measurement.record.independent>` does not let you indicate the `depends_on` field while :func:`dependent <labcore.measurement.record.dependent>`, has an empty list (indicating that it depends an all other independents) as a default.

>>> @recording(independent('x'), dependent('y', type='array'))
>>> def measure_stuff(n, *args, **kwargs):
>>>    return n, np.random.normal(size=n)
>>>
>>> measure_stuff(1)
{'x': 1, 'y': array([1.60113794])}

.. note::
    You can also use the abbreviations:

        * :class:`ds <labcore.measurement.record.ds>` for shorter :class:`DataSpec <labcore.measurement.record.DataSpec>`
        * :func:`indep() <labcore.measurement.record.indep>` for shorter :func:`independent <labcore.measurement.record.independent>`
        * :func:`dep() <labcore.measurement.record.dep>` for shorter :func:`dependent <labcore.measurement.record.dependent>`

Sometimes we don't want to annotate a function or generator itself, but instead we want to annotate at the moment of execution.
For this we can use the function :func:`record_as() <labcore.measurement.record.record_as>` to annotate any function or generator on the fly:

>>> def get_some_data(n):
>>>     return np.random.normal(size=n)
>>>
>>> record_as(get_some_data, independent('random_var'))(3)
{'random_var': array([0.16099358, 0.74873271, 0.01160423])}

You can add multiple :class:`DataSpecs <labcore.measurement.record.DataSpec>` with in a single :func:`record_as() <labcore.measurement.record.record_as>`:

>>> for data in record_as(zip(np.linspace(0,1,3), np.arange(3)), indep('x'), dep('y')):
>>>     print(data)
{'x': 0.0, 'y': 0}
{'x': 0.2, 'y': 1}
{'x': 0.4, 'y': 2}

It will also make sure to add items for annotated **records** (by adding `None` items to any empty **record**) that do not have any values assigned to them:

>>> for data in record_as(np.linspace(0,1,3), indep('x'), dep('y')):
>>>     print(data)
{'x': 0.0, 'y': None}
{'x': 0.5, 'y': None}
{'x': 1.0, 'y': None}

And it will ignore any extra values that are not annotated:

>>> for data in record_as(zip(np.linspace(0,1,3), np.arange(3)), indep('x')):
>>>     print(data)
{'x': 0.0}
{'x': 0.5}
{'x': 1.0}

Construction of Sweeps
----------------------

Now that we know how to annotate data so that it generates records, we can finally start creating a Sweep that creates some data.
A Sweep is composed of two main parts: **pointers** and **actions**.
**Pointers** are iterables that the sweep iterates through, these usually represent the independent variables of our experiments.
**Actions** are callables that get called after each iteration of our **pointer** and usually are in charge of performing anything that needs to happen at every iteration of the experiment.
This can be either set up a instruments and usually includes measuring a dependent variable too.
Both **pointers** and **actions** can generate **records** if annotated correctly, but it is not a requirement.

Basic Sweeps
^^^^^^^^^^^^

A basic annotated Sweep looks something like this:

>>> def my_func():
>>>     return 0
>>>
>>> sweep = Sweep(
>>>     record_as(range(3), independent('x')), # This is the pointer. We specify 'x' as an independent (we control it).
>>>     record_as(my_func, dependent('y'))) # my_func is an action. We specify 'y' as a dependent.

Once the Sweep is created we can see the **records** it will produce by utilising the function method :meth:`get_data_specs() <labcore.measurement.sweep.Sweep.get_data_specs>`:

>>> sweep.get_data_specs()
(x, y(x))

Printing a Sweep will also display more information about, specifying the pointers, the actions taken afterwards and the **records** it will produce:

>>> print(sweep)
range(0, 3) as {x} >> my_func() as {y}
==> {x, y(x)}

Now to run the Sweep we just have to iterate through it:

>>> for data in sweep:
>>>     print(data)
{'x': 0, 'y': 0}
{'x': 1, 'y': 0}
{'x': 2, 'y': 0}

If you are trying to sweep over a single parameter, a more convenient syntax for creating Sweep is to utilize the :func:`sweep_parameter() <labcore.measurement.sweep.sweep_parameter>` function:

>>> sweep = sweep_parameter('x', range(3), record_as(my_func, 'y'))
>>> for data in sweep:
>>>     print(data)
{'x': 0, 'y': 0}
{'x': 1, 'y': 0}
{'x': 2, 'y': 0}

There is no restriction on how many parameters a **pointer** or an **action** can generate as long as each parameter is properly annotated.

>>> def my_func():
>>>     return 1, 2
>>>
>>> sweep = Sweep(
>>>     record_as(zip(range(3), ['a', 'b', 'c']), independent('number'), independent('string')), # a pointer with two parameters
>>>     record_as(my_func, 'one', 'two'))
>>>
>>> print(sweep.get_data_specs())
>>>
>>> for data in sweep:
>>>     print(data)
(number, string, one(number, string), two(number, string))
{'number': 0, 'string': 'a', 'one': 1, 'two': 2}
{'number': 1, 'string': 'b', 'one': 1, 'two': 2}
{'number': 2, 'string': 'c', 'one': 1, 'two': 2}

Specifying Options Before Executing a Sweep
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Many **actions** we are using take optional parameters we only want to specify just before executing the Sweep (but are constant throughout the Sweep).

If we don't want to resort to global variables we can do so by using the method :meth:`set_options() <labcore.measurement.sweep.Sweep.set_options>`.
It accepts the names of any action functions in that Sweep as keywords, and dictionaries containing keyword arguments to pass to those functions as value.
Keywords specified in this way always override key words that are passed around internally in the sweep (**HERE HERE HERE HERE HERE HERE HERE HERE HERE HERE HERE lINK TO SPECIFIED SECTION LATER**):

>>> def test_fun(a_property=False, *args, **kwargs):
>>>     print('inside test_fun:')
>>>     print(f"a_property: {a_property}")
>>>     print(f"other stuff:", args, kwargs)
>>>     print('----')
>>>     return 0
>>>
>>> sweep = sweep_parameter('value', range(3), record_as(test_fun, dependent('data')))
>>> sweep.set_options(test_fun=dict(a_property=True, another_property='Hello'))
>>>
>>> for data in sweep:
>>>     print("Data:", data)
>>>     print('----')
inside test_fun:
property: True
other stuff: () {'value': 0, 'another_property': 'Hello'}
----
Data: {'value': 0, 'data': 0}
----
inside test_fun:
property: True
other stuff: () {'value': 1, 'data': 0, 'another_property': 'Hello'}
----
Data: {'value': 1, 'data': 0}
----
inside test_fun:
property: True
other stuff: () {'value': 2, 'data': 0, 'another_property': 'Hello'}
----
Data: {'value': 2, 'data': 0}


A QCoDeS Parameter Sweep
^^^^^^^^^^^^^^^^^^^^^^^^

If you are using QCoDeS to interact with hardware, it is very common to want to do a sweep over a QCoDeS parameter.
In this minimal example we set a parameter (``x``) to a range of values, and get data from another parameter for each set value.

>>> def measure_stuff():
>>>     return np.random.normal()
>>>
>>> x = Parameter('x', set_cmd=lambda x: print(f'setting x to {x}'), initial_value=0) # QCoDeS Parameter
>>> data = Parameter('data', get_cmd=lambda: np.random.normal()) # QCoDeS Parameter
>>>
>>> for record in sweep_parameter(x, range(3), get_parameter(data)):
>>>     print(record)
setting x to 0
setting x to 0
{'x': 0, 'data': -0.4990053668503893}
setting x to 1
{'x': 1, 'data': -0.5132204673887943}
setting x to 2
{'x': 2, 'data': 1.8634243556469932}

Sweep Combinations
------------------

One of the most valuable features of Sweeps is their ability to be able to combine them through the use of operators.
This allows us to mix and match different aspects of an experiment without having to rewrite code.
We can combine different Sweeps with each other or different annotated **actions** (**actions** that produce **records**).

Appending
^^^^^^^^^

The most basic combination of Sweeps is appending them.
When appending two Sweeps, the resulting sweep will execute the first Sweep to completion followed by the second Sweep to completion.
To append two Sweeps or actions we use the `+` symbol:

>>> def get_random_number():
>>>     return np.random.rand()
>>>
>>> Sweep.record_none = False # See note on what this does.
>>>
>>> sweep_1 = sweep_parameter('x', range(3), record_as(get_random_number, dependent('y')))
>>> sweep_2 = sweep_parameter('a', range(4), record_as(get_random_number, dependent('b')))
>>> my_sweep = sweep_1 + sweep_2
>>> for data in my_sweep:
>>>     print(data)
{'x': 0, 'y': 0.34404570192577155}
{'x': 1, 'y': 0.02104831292457654}
{'x': 2, 'y': 0.9006367857458307}
{'a': 0, 'b': 0.10539935409724577}
{'a': 1, 'b': 0.9368463758729733}
{'a': 2, 'b': 0.9550070757291859}
{'a': 3, 'b': 0.9812445448108895}

.. note::
    :meth:`Sweep.return_none <labcore.measurement.sweep.Sweep.return_none>` controls whether we include data fields that have returned nothing during setting a pointer or executing an action.
    Setting it to true (the default) guarantees that each data spec of the sweep has an entry per sweep point, even if it is ``None``.

Multiplying
^^^^^^^^^^^

By multiplying we refer to an inner product, i.e. the result is what you'd expect from `zip <https://docs.python.org/3.3/library/functions.html#zip>`__-ing two iterables.
To multiply two Sweeps or actions we use the `*` symbol.
A basic example is if we have a sweep and want to attach another action to each sweep point:

>>> my_sweep = (
>>>     sweep_parameter('x', range(3), record_as(get_random_number, dependent('data_1')))
>>>     * record_as(get_random_number, dependent('data_2'))
>>> )
>>>
>>> print(sweep.get_data_specs())
>>> print('----')
>>>
>>> for data in my_sweep:
>>>     print(data)
(x, data_1(x), data_2(x))
----
{'x': 0, 'data_1': 0.12599818360565485, 'data_2': 0.09261266841087679}
{'x': 1, 'data_1': 0.5665798938860637, 'data_2': 0.7493750740615404}
{'x': 2, 'data_1': 0.9035085438172156, 'data_2': 0.5419023528195611}

If you are combining two different Sweeps, then we get zip-like behavior and the dependency structure remains separate:

>>> my_sweep = (
>>>     sweep_parameter('x', range(3), record_as(get_random_number, dependent('data_1')))
>>>     * sweep_parameter('y', range(5), record_as(get_random_number, dependent('data_2')))
>>> )
>>>
>>> print(sweep.get_data_specs())
>>> print('----')
>>>
>>> for data in my_sweep:
>>>     print(data)
(x, data_1(x), y, data_2(y))
----
{'x': 0, 'data_1': 0.3808452915069015, 'y': 0, 'data_2': 0.14309246334791337}
{'x': 1, 'data_1': 0.6094608905204076, 'y': 1, 'data_2': 0.3560530722571186}
{'x': 2, 'data_1': 0.15950240245080072, 'y': 2, 'data_2': 0.2477391943438858}

Nesting
^^^^^^^

Nesting two Sweeps runs the entire second Sweep for each point of the first Sweep.
A basic example is if we have multiple Sweep parameters against each other and we want to perform a measurement at each point.
To nest two Sweeps we use the `@`:

>>> def measure_something():
>>>     return np.random.rand()
>>>
>>> my_sweep = (
>>>     sweep_parameter('x', range(3))
>>>     @ sweep_parameter('y', np.linspace(0,1,3))
>>>     @ record_as(measure_something, 'my_data')
>>> )
>>>
>>> for data in my_sweep:
>>>     print(data)
{'x': 0, 'y': 0.0, 'my_data': 0.727404046865409}
{'x': 0, 'y': 0.5, 'my_data': 0.11112429412122715}
{'x': 0, 'y': 1.0, 'my_data': 0.09081900115421426}
{'x': 1, 'y': 0.0, 'my_data': 0.8160224024098803}
{'x': 1, 'y': 0.5, 'my_data': 0.1517092154216605}
{'x': 1, 'y': 1.0, 'my_data': 0.9253018251769569}
{'x': 2, 'y': 0.0, 'my_data': 0.881089486629102}
{'x': 2, 'y': 0.5, 'my_data': 0.3897577898200387}
{'x': 2, 'y': 1.0, 'my_data': 0.6895312744116066}

Nested sweeps can be as complex as needed, with as many actions as they need.
An example of this can be executing measurements on each nested level:

>>> def measure_something():
>>>     return np.random.rand()
>>>
>>> sweep_1 = sweep_parameter('x', range(3), record_as(measure_something, 'a'))
>>> sweep_2 = sweep_parameter('y', range(2), record_as(measure_something, 'b'))
>>> my_sweep = sweep_1 @ sweep_2 @ record_as(get_random_number, 'more_data')
>>>
>>> for data in my_sweep:
>>>     print(data)
{'x': 0, 'a': 0.09522178419462424, 'y': 0, 'b': 0.1821505218348034, 'more_data': 0.13257002268089835}
{'x': 0, 'a': 0.09522178419462424, 'y': 1, 'b': 0.014940266372080457, 'more_data': 0.9460879863404558}
{'x': 1, 'a': 0.13994892182170526, 'y': 0, 'b': 0.4708657480125388, 'more_data': 0.12792337523097086}
{'x': 1, 'a': 0.13994892182170526, 'y': 1, 'b': 0.8209492135277935, 'more_data': 0.23270477191895111}
{'x': 2, 'a': 0.06159208933324678, 'y': 0, 'b': 0.651545802505077, 'more_data': 0.8944257582518365}
{'x': 2, 'a': 0.06159208933324678, 'y': 1, 'b': 0.9064557565446919, 'more_data': 0.8258102740474211}

.. note::
    All operators symbols are just there for syntactic brevity.
    All three of them have corresponding functions attached to them:
       * Appending: :func:`append_sweeps() <labcore.measurement.sweep.append_sweeps>`
       * Multiplying: :func:`zip_sweeps() <labcore.measurement.sweep.zip_sweeps>`
       * Nesting: :func:`nest_sweeps() <labcore.measurement.sweep.nest_sweeps>`

Passing Parameters in a Sweep
-----------------------------

Often times our measurement actions depend on the states of previous steps.
Because of that, everything that is generated by **pointers**, **actions** or other Sweeps can be passed on subsequently executed elements.

.. note::
    There are two different Sweep configuration related to passing arguments in Sweeps. For more information on them see the :ref:`sweep_configuration` part of this guide.

Positional Arguments
^^^^^^^^^^^^^^^^^^^^

When there is no record annotations, the values generated **only** by **pointers** are passed as positional arguments to all actions, but values generated by **actions** are not passed to other **actions**:

>>> def test(*args, **kwargs):
>>>     print('test:', args, kwargs)
>>>     return 101
>>>
>>> def test_2(*args, **kwargs):
>>>     print('test_2:', args, kwargs)
>>>     return 102
>>>
>>> for data in Sweep(range(3), test, test_2):
>>>     print(data)
test: (0,) {}
test_2: (0,) {}
{}
test: (1,) {}
test_2: (1,) {}
{}
test: (2,) {}
test_2: (2,) {}
{}

Because it would get too confusing otherwise, positional arguments only get passed originating from a **pointer** to all **actions** in a single sweep.
Meaning that if we combine two or more sweeps, positional arguments would only get to the **actions** of their respective Sweeps:

>>> for data in Sweep(range(3), test) * Sweep(zip(['x', 'y'], [True, False]), test):
>>>    print(data)
(0,) {}
('x', True) {}
{}
(1,) {}
('y', False) {}
{}
(2,) {}

As we can see the `test` function in the second sweep is only getting (`x`, `True`) or (`y`, `False`) but not any arguments from the first Sweep.
It is also important to note that hte values generated by either `test` function are not being passed to any other object.

In previous examples, the functions we used were accepting the arguments because their signature included variation positional arguments (`*args`). The situation changes when this is not the case.
**Actions** only receive arguments that they can accept:

>>> def test_3(x=10):
>>>     print(x)
>>>     return True
>>>
>>> for data in Sweep(zip([1,2], [3,4]), test_3):
>>>     pass
1
2

As we can see, `test_3` only accepted the first argument.

Keyword Arguments
^^^^^^^^^^^^^^^^^

Passing keyword arguments is more flexible.
Any **record** (annotated data) that gets produced gets passed to all subsequent functions in the sweep, **pointers** or **actions**, that accept that keyword.
This is true even across different sub-sweeps
If a **pointer** yields non-annotated values, these are still used as positional arguments, but only when accepted, and with higher priority given to keywords.
In the following example we can see this in action:

>>> def test(x, y, z=5):
>>>     print(f'my three arguments, x: {x}, y: {y}, z: {z}')
>>>     return x, y, z
>>>
>>> def print_all_args(*args, **kwargs):
>>>     print(f'arguments at the end of the line, args: {args}, kwargs: {kwargs}')
>>>
>>> sweep = sweep_parameter('x', range(3), record_as(test, dep('xx'), dep('yy'), dep('zz'))) * Sweep(range(3), print_all_args)
>>> for data in sweep:
>>>     pass
my three arguments, x: 0, y: None, z: 5
arguments at the end of the line, args:(0,), kwargs:{'x': 0, 'xx': 0, 'zz': 5}
my three arguments, x: 1, y: None, z: 5
arguments at the end of the line, args:(1,), kwargs:{'x': 1, 'xx': 1, 'zz': 5}
my three arguments, x: 2, y: None, z: 5
arguments at the end of the line, args:(2,), kwargs:{'x': 2, 'xx': 2, 'zz': 5}

In the example above we have two different sweeps.
The **pointer** of the first one is producing **records** which is why we its value in the test function for `x`.
Since the first sweep is being multiplied to the second sweep we can see how all the **records** (both produce by the **pointer** and **action**) of the first sweep reach as keyword arguments, and the non-annotated value of its own **pointer** reaches the action of the second Sweep as a positional argument.

.. warning::
    When creating **records**, it is very important that each **record** has a *unique* name. Having multiple variables
    create **records** with the same names, will make the passing or arguments behave in unpredictable ways.

A simple way of renaming conflicting arguments and **records** is to us the combination of `lambda` and :func:`record_as() <labcore.measurement.record.record_as>`:

>>> sweep = (
>>>     Sweep(record_as(zip(range(3), range(10,13)), independent('x'), independent('y')), record_as(test, dependent('xx'), dependent('yy'), dependent('zz')))
>>>     @ record_as(lambda xx, yy, zz: test(xx, yy, zz), dependent('some'), dependent('different'), dependent('names'))
>>>     @ print_all_args
>>>     + print_all_args)
>>>
>>> print(sweep.get_data_specs())
>>>
>>> for data in sweep:
>>>     print("data:", data)
(x, y, xx(x, y), yy(x, y), zz(x, y), some(x, y), different(x, y), names(x, y))
my three arguments: 0 10 5
my three arguments: 0 10 5
arguments at the end of the line: () {'x': 0, 'y': 10, 'xx': 0, 'yy': 10, 'zz': 5, 'some': 0, 'different': 10, 'names': 5}
data: {'x': 0, 'y': 10, 'xx': 0, 'yy': 10, 'zz': 5, 'some': 0, 'different': 10, 'names': 5}
my three arguments: 1 11 5
my three arguments: 1 11 5
arguments at the end of the line: () {'x': 1, 'y': 11, 'xx': 1, 'yy': 11, 'zz': 5, 'some': 1, 'different': 11, 'names': 5}
data: {'x': 1, 'y': 11, 'xx': 1, 'yy': 11, 'zz': 5, 'some': 1, 'different': 11, 'names': 5}
my three arguments: 2 12 5
my three arguments: 2 12 5
arguments at the end of the line: () {'x': 2, 'y': 12, 'xx': 2, 'yy': 12, 'zz': 5, 'some': 2, 'different': 12, 'names': 5}
data: {'x': 2, 'y': 12, 'xx': 2, 'yy': 12, 'zz': 5, 'some': 2, 'different': 12, 'names': 5}
arguments at the end of the line: () {'x': 2, 'y': 12, 'xx': 2, 'yy': 12, 'zz': 5, 'some': 2, 'different': 12, 'names': 5}
data: {}

Configuring Sweeps
------------------

.. sweep_configuration:

The class :class:`Sweep <labcore.measurement.sweep.Sweep>` has three global parameters that are used to configure the
behaviour of it.

record_none
^^^^^^^^^^^

:class:`Sweep.record_none <labcore.measurement.sweep.Sweep.record_none>` ,`True` by default, adds `None` to any **action** or **pointer** that didn't generate any **record** that iteration.
This is useful if we want every variable we are storing be composed of arrays of the same number of items:

>>> def get_random_number():
>>>     return np.random.rand()

>>> sweep_1 = sweep_parameter('x', range(3), record_as(get_random_number, dependent('y')))
>>> sweep_2 = sweep_parameter('a', range(4), record_as(get_random_number, dependent('b')))
>>> my_sweep = sweep_1 + sweep_2

>>> Sweep.record_none = False
>>> print(f'----record_none=False----')
>>> for data in my_sweep:
>>>     print(data)
>>>
>>> Sweep.record_none = True
>>> print(f'----record_none=True----')
>>> for data in my_sweep:
>>>     print(data)
----record_none=False----
{'x': 0, 'y': 0.804635124804199}
{'x': 1, 'y': 0.24410055642545125}
{'x': 2, 'y': 0.10828652013926787}
{'a': 0, 'b': 0.4303128288315823}
{'a': 1, 'b': 0.9498154942316515}
{'a': 2, 'b': 0.7150406031589893}
{'a': 3, 'b': 0.2012281139956017}
----record_none=True----
{'x': 0, 'y': 0.22753548379033073, 'a': None, 'b': None}
{'x': 1, 'y': 0.9024597689210428, 'a': None, 'b': None}
{'x': 2, 'y': 0.11393941613249503, 'a': None, 'b': None}
{'x': None, 'y': None, 'a': 0, 'b': 0.8678669225696442}
{'x': None, 'y': None, 'a': 1, 'b': 0.3537275760737344}
{'x': None, 'y': None, 'a': 2, 'b': 0.23555393946522196}
{'x': None, 'y': None, 'a': 3, 'b': 0.19388827122308672}

pass_on_returns
^^^^^^^^^^^^^^^

:class:`Sweep.pass_on_returns <labcore.measurement.sweep.Sweep.pass_on_returns>` ,`True` by default, specifies if we want arguments to be passed between sweeps.
When it is set to `False` no **record** will be passed either as positional arguments or as keyword arguments:

>>> sweep = (
>>>     sweep_parameter('y', range(3), record_as(test, dependent('xx'), dependent('yy'), dependent('zz')))
>>>     @ print_all_args
>>> )
>>>
>>> Sweep.pass_on_returns = False
>>> print(f'----pass_on_returns=False----')
>>> for data in sweep:
>>>     print("data:", data)
>>>
>>>
>>> print()
>>> Sweep.pass_on_returns = True
>>> print(f'----pass_on_returns=True----')
>>> for data in sweep:
>>>     print("data:", data)
----pass_on_returns=False----
my three arguments: None None 5
arguments at the end of the line: () {}
data: {'y': 0, 'xx': None, 'yy': None, 'zz': 5}
my three arguments: None None 5
arguments at the end of the line: () {}
data: {'y': 1, 'xx': None, 'yy': None, 'zz': 5}
my three arguments: None None 5
arguments at the end of the line: () {}
data: {'y': 2, 'xx': None, 'yy': None, 'zz': 5}

----pass_on_returns=True----
my three arguments: None 0 5
arguments at the end of the line: () {'zz': 5, 'y': 0, 'yy': 0}
data: {'y': 0, 'xx': None, 'yy': 0, 'zz': 5}
my three arguments: None 1 5
arguments at the end of the line: () {'zz': 5, 'y': 1, 'yy': 1}
data: {'y': 1, 'xx': None, 'yy': 1, 'zz': 5}
my three arguments: None 2 5
arguments at the end of the line: () {'zz': 5, 'y': 2, 'yy': 2}
data: {'y': 2, 'xx': None, 'yy': 2, 'zz': 5}

pass_on_none
^^^^^^^^^^^^

:class:`Sweep.pass_on_none <labcore.measurement.sweep.Sweep.pass_on_none>`, `False` by default, specifies if variables that return `None` should be passed as arguments to other **actions** or Sweeps ((Because `None` is typically indicating that function did not return anything as data even though a record was declared using `recording` or `record_as`).:

>>> sweep = (
>>>     sweep_parameter('y', range(3), record_as(test, dependent('xx'), dependent('yy'), dependent('zz')))
>>>     @ print_all_args)
>>>
>>> Sweep.pass_on_none = False
>>> print(f'----pass_on_none=False----')
>>> for data in sweep:
>>>     print("data:", data)
>>>
>>> print()
>>> Sweep.pass_on_none = True
>>> print(f'----pass_on_returns=True----')
>>> for data in sweep:
>>>     print("data:", data)
----pass_on_none=False----
my three arguments: None 0 5
arguments at the end of the line: () {'y': 0, 'yy': 0, 'zz': 5}
data: {'y': 0, 'xx': None, 'yy': 0, 'zz': 5}
my three arguments: None 1 5
arguments at the end of the line: () {'y': 1, 'yy': 1, 'zz': 5}
data: {'y': 1, 'xx': None, 'yy': 1, 'zz': 5}
my three arguments: None 2 5
arguments at the end of the line: () {'y': 2, 'yy': 2, 'zz': 5}
data: {'y': 2, 'xx': None, 'yy': 2, 'zz': 5}

----pass_on_returns=True----
my three arguments: None 0 5
arguments at the end of the line: (None,) {'y': 0, 'yy': 0, 'zz': 5, 'xx': None}
data: {'y': 0, 'xx': None, 'yy': 0, 'zz': 5}
my three arguments: None 1 5
arguments at the end of the line: (None,) {'y': 1, 'yy': 1, 'zz': 5, 'xx': None}
data: {'y': 1, 'xx': None, 'yy': 1, 'zz': 5}
my three arguments: None 2 5
arguments at the end of the line: (None,) {'y': 2, 'yy': 2, 'zz': 5, 'xx': None}
data: {'y': 2, 'xx': None, 'yy': 2, 'zz': 5}

Running Sweeps
--------------

As seen in previous examples, the most basic way of running a Sweep is to just iterate through it.
This is simple, but it also makes it such that what we do with the data is up to us.
Most of the time when running Sweeps, we want the data to be saved directly to disk.
For this the function :func:`run_and_save_sweep() <labcore.ddh5.run_and_save_sweep>` exists:

>>> sweep = sweep_parameter('x', range(3), record_as(my_func, 'y'))
>>> run_and_save_sweep(sweep, './data', 'my_data')
Data location:  data/2022-12-05/2022-12-05T142539_fbfce3e4-my_data/data.ddh5
The measurement has finished successfully and all of the data has been saved.

:func:`run_and_save_sweep() <labcore.ddh5.run_and_save_sweep>` automatically runs the Sweep indicated, stores the **records** generated by it in a :class:`DataDict <plottr.data.datadict.DataDict>` in a ddh5 file with time tag followed by a random sequence followed by the third argument, in the directory passed by the second argument.
Internally the function utilizes the :class:`DDH5Writer <plottr.data.datadict_storage.DDH5Writer>` from `plottr`.
For more information on how `plottr` handles data please see: :ref:`_datadict_documentation`.

.. note::
    :func:`run_and_save_sweep() <labcore.ddh5.run_and_save_sweep>` can be passed extra arguments to store different items.
    It is a good idea to read over its documentation if you want to be able to save things with it.

Sometimes we have an action that we want to run a single time, some kind of setup function or maybe a closing function.
If we also need this to be a Sweep, the function :func:`once() <labcore.measurement.sweep.once>` will create a Sweep with no **pointer** that runs an **action** a single time:

>>> def startup_function():
>>>     print(f'starting an instrument')
>>>
>>> def closing_function():
>>>     print(f'closing an instrument')
>>>
>>> sweep = sweep_parameter('x', range(3), record_as(my_func, 'y'))
>>> starting_sweep = once(startup_function)
>>> closing_sweep = once(closing_function)
>>>
>>> for data in starting_sweep + sweep + closing_sweep:
>>>     print(data)
starting an instrument
{}
{'x': 0, 'y': 1}
{'x': 1, 'y': 1}
{'x': 2, 'y': 1}
closing an instrument
{}

Reference
---------

Sweep
^^^^^

.. automodule:: labcore.measurement.sweep
    :members:

Record
^^^^^^

.. automodule:: labcore.measurement.record
    :members:

ddh5
^^^^

.. automodule:: labcore.ddh5
    :members:






Basic Usage
===========

Installation
------------

At the moment `instrumentserver` is not on pip or conda so the only way of installing it is to install it from github directly.
To do that first clone the `github repo <https://github.com/toolsforexperiments/labcore>`__,
and install into the desired environment using the
`editable pip install <https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-e>`_.

Quick Overview
--------------

Concepts
^^^^^^^^

The main task for running experiemtns utlizing `labcore` is to define what your :class:`Sweep <labcore.measurement.Sweep>` is.
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

Where the `range` function is your **pointer** and the 2 functions are your **actions**.

Once the sweep is created but before execution we can easily infer what records the sweep produces.
Sweeps can be combined in ways that result in nesting, zipping, or appending. Combining sweeps again results in a sweep.
This will allow us to construct modular measurements from pre-defined blocks.
For more information please see :ref:`Introduction to sweeping <labcore introduction to sweeping>`









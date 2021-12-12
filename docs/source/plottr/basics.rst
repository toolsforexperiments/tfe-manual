Basic usage
===========

Installation
------------

There are multiple options:

Recent stable versions can be installed either with pip (``$ pip install plottr``)
or conda, using the conda-forge channel (``$ conda install -c conda-forge plottr``).

If you want to play with the most recent versions, or contribute to the development,
it makes sense to install from github directly.
In that case, clone the `github repo <https://github.com/toolsforexperiments/plottr>`_,
and install into the desired environment using the
`editable pip install <https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-e>`_.

.. _essential tools:

Essential tools for inspecting data
-----------------------------------

There are a few different ways of easy data inspection using tools that come
predefined with the basic plottr installation.

- interactive use from IPython
- loading data from a `QCoDeS` database
- loading data from HDF5 files

In the following we briefly introduce all of these.
We will use the `autoplot` app that comes with plottr by default.

.. note::
    The examples below are also included in the notebook ``doc/examples/Plottr quickstart.ipynb``
    that comes with the plottr repository.


Interactive use
^^^^^^^^^^^^^^^

The easiest way to inspect data with plottr via the `autoplot app` is to use
the :func:`plottr.apps.autoplot.autoplot` function from IPython or Jupyter.


Loading QCoDeS data
^^^^^^^^^^^^^^^^^^^

TBD.

Loading data from HDF5
^^^^^^^^^^^^^^^^^^^^^^

TBD.

Live plotting measurement data
------------------------------

TBD.









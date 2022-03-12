.. Tools for Experiments Manual documentation master file, created by
   sphinx-quickstart on Fri Aug 13 15:51:32 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Tools for Experiments
=====================

`Tools for experiments` is a collection of Python packages that are primarily
designed for running and analyzing experiments in physics laboratories.
The software is meant to provide some additional convenience and capability on
top of libraries that provide access to measurement hardware,
such as `qcodes <https://qcodes.github.io/Qcodes/>`_.
While the workflow in mind is electronic measurements of quantum devices
(such as superconducting qubits or semiconductor quantum dots) it may well be
useful for a much broader range of experiment needs.

Tools for experiments currently consists of the packages listed below.
Each tool can be used independently (but they do play well together!).

.. todo::
    we should add a few screenshots or snippets that illustrate how some of the tools work.

plottr
------

`Plottr` is a GUI tool for inspecting and monitoring data.
The primary use case for it is inspecting measurement data (incl live-plotting
while data is being acquired).
Plottr allows easy graphical inspection of multidimensional data.
Plotting can be supplemented with custom data analysis and processing
that can be integrated into the GUI.

instrumentserver
----------------

`instrumentserver` is a program that runs qcodes instruments in a dedicated
server program such that client processes can access them.
This allows, for example, accessing the same instrument from multiple processes


labcore
-------

This package contains a set of convenience tools that make setting up
measurements easier.


.. toctree::
   :maxdepth: 2
   :glob:
   :caption: Contents:

   plottr/index

..   api/index

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

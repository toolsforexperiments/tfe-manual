Plottr
======

This is the main documentation for the data analysis tool `plottr`.
`plottr` is written in python (version 3.7+) and has been tested to work well on Windows, Linux,
and MacOS.
This documentation is still very much ongoing work in progress, but should (hopefully!)
already give an overview of what it's about, and how to use it.

The aim of `plottr` is to provide a simple but powerful graphical tool that allows efficient
inspection of measurement data as frequently found in experimental physics labs (but it
is in no way confined to that use).
In particular, it allows to define analysis flows that process data (which could come
from a file, or some other source) to very easily produce plots that give insight
into the data.
The goal is not to produce publication-level figures, but rather to be able to
quickly 'dissect' or analyze complicated and often multi-dimensional measurement
data to determine the best next course of action.
For some basic examples, please see :ref:`the basic usage page <essential tools>`.

In addition, `plottr` is written explicitly with the goal in mind to be extendable.
It is possible to define custom analysis flows that the users can implement themselves.
Any kind of data manipulation or analysis (from simple things such as subtracting offsets,
to more complex things like automatic fitting) can be added by the user.

Plottr is usable and pretty stable at this moment.
However, we're still working on quite a few features, so new things are being improved
and changed a lot. Please check back regularly to see what's new!


..  toctree::
    :maxdepth: 2
    :caption: Contents

    basics
    data
    apps
    monitr






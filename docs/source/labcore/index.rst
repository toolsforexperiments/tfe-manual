Labcore
=======

This is the main documentation for the experiment control tools `labcore`.
`labcore` is written in python and has been tested to work well on Windows, Linux,
and MacOS.
This documentation is still very much ongoing work in progress, but should (hopefully!)
already give an overview of what it's about, and how to use it.

The aim of `labcore` is to provide tools to facilitate the modularization of repetition of experiment control software.
`Labcore` allows you to separate different actions of an experiment and modularizing it, allowing you to mix and match
different parts or entire parts of experiments.
Having this structure around experiments allow you to have access to main characteristics (such as the type of data acquired during the run) in advance,
before any measurement code is executed.


..  toctree::
    :maxdepth: 2
    :caption: Contents

    basics
    intro
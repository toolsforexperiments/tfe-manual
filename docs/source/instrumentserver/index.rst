Instrumentserver
================

This is the main documentation for the instrument management tool `instrumentserver`.
`instrumentserver` is written in python and has been tested to work well on Windows, Linux,
and MacOS.
This documentation is still very much ongoing work in progress, but should (hopefully!)
already give an overview of what it's about, and how to use it.

The aim of `instrumentserver` is to facilitate `QCoDeS <https://qcodes.github.io/Qcodes/>`__ access across a variety of process and devices.
We communicate with the server through a TCP/IP connection allowing us to talk to it from any independent process or
separate device in the same network.

`instrumentserver` also includes a virtual instrument called `parameter manager` whose job is to be a centralized and
single source of truth for various parameters values with a user friendly graphical interface to facilitate changing
parameters.
For more information, please see

..  toctree::
    :maxdepth: 2
    :caption: Contents

    basics
    code

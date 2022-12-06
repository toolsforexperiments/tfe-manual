Basic Usage
===========

Installation
------------

At the moment `instrumentserver` is not on pip or conda so the only way of installing it is to install it from github directly.
To do that first clone the `github repo <https://github.com/toolsforexperiments/instrumentserver>`__,
and install into the desired environment using the
`editable pip install <https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-e>`_.

.. _instrumentserver essential tools:

Quick Overview
--------------

Instrumentserver
^^^^^^^^^^^^^^^^

To open the instrument server we simple run the command on a terminal:

.. code-block:: console

    $ instrumentserver

This will open the GUI of the server and start running it.

.. image:: img/empty_server.png

.. note::
    The server can be run without a gui by passing the --gui False argument.

By default, instrumentserver listens to the local host IP address (127.0.0.1) and the port 5555. To be able to communicate
with the server through other devices in the network we have to specify the IP address we want the server to listen to.
For this we pass the argument -a <IP_address> and -p <port_number>:

.. code-block:: console

    $ instrumentserver -a 192.168.1.1 -p 12345

This will make the server listen to both the local host and the IP address 192.168.1.1 with port 12345.

We communicate with the server with Python code. This can be done anywhere that python can run, an IPython console, a Jupyter notebook, etc.
The easiest way of creating :class:`Client <instrumentserver.client.proxy.Client>` and
running the :func:`find_or_create_instrument() <instrumentserver.client.proxy.Client.find_or_create_instrument>` method.

.. note::
    Remember to pass the instrument specific arguments and keyword arguments necessary for the specific QCoDeS instrument
    you are trying to open.

This will look for the specified instrument with the given name in the server or create it if the instrument does not
exist, and return it:

>>> cli = Client()
>>> dummy_instrument = cli.create_instrument(instrument_class='instrumentserver.testing.dummy_instruments.generic.DummyChannel', name='dummy_instrument')

.. note::
    If we are trying to talk to a server running in a different device in the network we need to specify the IP address
    and port with the arguments host and port when creating the :class:`Client <instrumentserver.client.proxy.Client>`.

After this we can see that the instrument has been created in the server.

.. image:: img/dummy_instrument.png

After that we can use the instrument like a normal QCoDeS instrument. We can create a :class:`Client <instrumentserver.client.proxy.Client>`
from any process and get the dummy_instrument by simply using the :func:`get_instrument() <instrumentserver.client.proxy.Client.get_instrument>` method:

>>> dummy_instrument = cli.get_instrument(name='dummy_instrument')

Parameter Manager
^^^^^^^^^^^^^^^^^

`instrumentserver` also comes with the virtual instrument Parameter Manager.
The Parameter Manager allows us to store values in an instrument inside of the `instrumentserver`, allowing us to access them from any process or devices in the same network.
The idea of it is to have a single source of truth for parameters whose values change frequently, and it provides a GUI from which you can change the values and easily see what they are.

To open the Parameter Manager we first need to open the `instrumentserver`.
Once we have the server open, we can run the command:

.. code-block:: console

    $ instrumentserver-param-manager

This will create an instance of the virtual instrument in the `instrumentserver` and will open the GUI for the Parameter Manager.

.. note::
    At the moment the parameter manager can only be opened from the device that is currently hosting the server.
    If you are utilizing a different port, this can be specified by passing the terminal argument `--port` followed by the port.

We'll simply get an empty window now.
The bottom of the window allows us to add arbitrary parameters and values, where dots serve as hierarchy separators (like objects and their children in python).

.. image:: img/parameter_manager.png

We can add some parameters and then retrieve them from anywhere that can run python code:

>>> cli = Client()
>>> params = cli.get_instrument('parameter_manager') # 'parameter_manager` is the name the startup script gives the instrument by default
>>> params.qubit.pipulse.len()
40

We can change parameters by calling the same function but passing as an argument the new value:

>>> params.qubit.pipulse.len(789)

We can add or remove parameters with code too:

>>> params.add_parameter('qubit.anharmonicity', initial_value=-150.0, unit='MHz')
>>> params.remove_parameter('qubit.pipulse.amp')

All of these changes get updated live in the GUI:

.. image:: img/updated_manager.png

Changing things in the GUI will also be reflected in the code.

.. warning::
    Changing something from the GUI only changes the code if we are calling the parameter manager directly.
    If we store a value in a separate variable and then change the GUI, the value in the variable might not get update.
    Because of this, we always recommend to call the Parameter Manager directly instead of saving the values in variables.

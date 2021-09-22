Files Created by "Tools for Experiments Overview"
=================================================

The Tools for Experiments Overview notebook creates two different json files utilized to save the values of the
instrumentserver and parameter manager to disk for later use. The specific files created are called "parameters.json"
and "parameters-with-units.json". The following tables are a visual representation of the json dictionary they contain:

Parameters.json
---------------

.. code-block:: json

    {
      "fluxctrl.flux": 0.45,
      "fluxctrl.inductive_participation_ratio": 0.05,
      "parameter_manager.qubit.anharmonicity": -150.0,
      "parameter_manager.qubit.frequency": 5.678,
      "parameter_manager.qubit.pipulse.amp": 0.4,
      "parameter_manager.resonator.flux": 0.2,
      "parameter_manager.resonator.frequency": 3,
      "vna.bandwidth": 10000.0,
      "vna.data": null,
      "vna.frequency": null,
      "vna.input_attenuation": 70,
      "vna.noise_temperature": 4.0,
      "vna.npoints": 1201,
      "vna.power": -50,
      "vna.resonator_frequency": 3000000000.0,
      "vna.resonator_linewidth": 10000000.0,
      "vna.start_frequency": 6089000000.0,
      "vna.stop_frequency": 7089000000.0
    }

parameters-with-units.json
--------------------------

.. code-block:: json

    {
      "parameter_manager.qubit.anharmonicity": {
        "unit": "MHz",
        "value": -150.0
      },
      "parameter_manager.qubit.frequency": {
        "unit": "GHz",
        "value": 5.678
      },
      "parameter_manager.qubit.pipulse.amp": {
        "unit": "V",
        "value": 0.4
      },
      "parameter_manager.resonator.flux": {
        "unit": "",
        "value": 0.2
      },
      "parameter_manager.resonator.frequency": {
        "unit": "GHz",
        "value": 3
      }
    }
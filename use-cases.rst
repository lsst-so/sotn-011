
.. note::

   Use-cases studies goes here.


.. _mtmount-telemetry:

MTMount Telemetry
-----------------

The source of truth for the MTMount CSC telemetry interface is an ``.ini`` file used by the TMA LabVIEW Controller to define system telemetry.
Currently, this file is translated to XML via a script in the ts_mtmount package.
We can leverage this same script to translate the ``.ini`` file directly into Avro schemas instead.

However, one caveat is that some telemetry in this interface is currently defined using arrays.
To handle these cases, the generated Avro topics will use single-element scalar fields instead of arrays, accompanied by an explicit index field.
The index field name could be configurable per topic, defaulting to something like ``sensorId``.

It is worth noting that while :cite:`SQR-113` classifies MTMount as having "packed data", this is actually a case of multi-sensor measurements.
Therefore, relying on timestamps alone to differentiate measurements would be incorrect.

.. _ess-case-study:

ESS Case Study
--------------

Use the ESS as a case study.

.. _m1m3-case-study:

M1M3 Case Study
---------------

M1M3 relies heavily in arrays for all classes of topics; telemetry, events and commands.
Of its 139 topics, 56 (around 40\%) contain array fields.

In most cases these arrays are used to publish information for different actuators in a compact way.
For example, the ``appliedForces`` telemetry contain 3 array fields; ``xForces``, ``yForces`` and ``zForces``, each containing the force applied on the individual actuators on the different axis.
However, it is also important to note that the force actuators have different geometry so no all of them acts on all axis.
This means that each of the fields mentioned above have different sizes; ``xForces`` contains 12 elements, ``yForces`` 100 and ``zForces`` 156.
Furthermore, the same topic also contains single value fields; ``fx``, ``fy``, ``fz``, ``mx``, ``my``, ``mz``, ``forceMagnitude``, a ``timestamp`` and all the common SAL fields.
These individual fields represent the bulk force and resulting momentum breakdown resulting from the arrays.
Having all this information compiled in a single topic considerably simplifies the consumption of this information on the control system side, and also on the analysis side.

Breaking down this topic into individual measurements would need 4 topics:

- ``appliedForces`` containing all the single value fields.
- ``appliedXForces`` containing the ``xForces`` values, a timestamp and an actuator id.
- ``appliedYForces`` containing the ``yForces`` values, a timestamp and an actuator id.
- ``appliedZForces`` containing the ``zForces`` values, a timestamp and an actuator id.

We could also potentially include a ``measurementId`` to all the aforementioned topics to help combine the information.

One obvious caveat to this change is that now, instead of being able to rely on a single topic to collect all the information, users will be required to subscribe and run expensive matching operation to rebuild the same state.
Note that a similar operation is currently executed in the MTAOS involving the ``appliedActiveOpticForces`` event.

For commands, arrays plays an even more important role as they allow setting force values for all actuators at once; which is critical for AOS operations.
This is currently done on the AOS side using the ``applyActiveOpticForces``, which receives an array of ``zForces`` to be applied to the mirror.
If we were to break this down into single values operation, it would also require splitting the operation into 2 stages:

- set the forces for all actuators but not apply it,
- apply the forces.

This means that an operation that is currently performed in a single command, would require 157 commands to be completed; 156 to set the forces for each actuator and 1 additional command to apply them.

Overall, this seems like a strong argument to maintain support for use of arrays in our interfaces.
Instead of removing arrays completely we could institute a more strict policy on where arrays can be used, limiting it to those use-cases where they represent a clear benefit from splitting the data into single measurements.
We could also investigate if there is any way we could improve how these array data are inserted into influxDB to improve compatibility.


.. _m2-case-study:

M2 Case Study
--------------

We need to look at the M1M3 and M2 interfaces as well. These (along with MTMount) are our system heavy hitters

.. _atmcs-case-study:

ATMCS Case Study
----------------

We need to consider the ATMCS as well.
The reason to have that time-packed telemetry is to support the 100 Hz telemetry.
That means if we do not use the array, we need to make sure we can publish the topics in 100 Hz.

WvR: The telemetry is time packed because LabVIEW struggles (to keep it friendly) with publishing at 100 Hz.
The CSC probably needs to unpack the telemetry received from the cRIO and publish that.

.. _hvac-and-mtdome:

HVAC and MTDome
---------------

HVAC and MTDome have Python code that (in case of MTDome partially) creates the XML so those will need to be rewritten to create avro instead.


.. _renaming-hvac-topic-attributes:

Renaming HVAC topic attributes
------------------------------

There is a proposal to rename many topic items in the HVAC (OSW-2192).
This is probably the time to do it since (as Michael suggested) we will create a new database.

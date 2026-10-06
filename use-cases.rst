
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

We need to look at the M1M3 and M2 interfaces as well. These (along with MTMount) are our system heavy hitters

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

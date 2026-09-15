
.. note::

   Use-cases studies goes here.


MTMount Telemetry
-----------------

Te-Wei just reminded me of the MTMount telemetry; which is translated into xml from an ini file used by the TMA controller.

ESS Case Study
--------------

Use the ESS as a case study.

M1M3 Case Study
---------------

We need to look at the M1M3 and M2 interfaces as well. These (along with MTMount) are our system heavy hitters

M2 Case Study
--------------

We need to look at the M1M3 and M2 interfaces as well. These (along with MTMount) are our system heavy hitters

ATMCS Case Study
----------------

We need to consider the ATMCS as well.
The reason to have that time-packed telemetry is to support the 100 Hz telemetry.
That means if we do not use the array, we need to make sure we can publish the topics in 100 Hz.

WvR: The telemetry is time packed because LabVIEW struggles (to keep it friendly) with publishing at 100 Hz.
The CSC probably needs to unpack the telemetry received from the cRIO and publish that.

HVAC and MTDome
---------------

HVAC and MTDome have Python code that (in case of MTDome partially) creates the XML so those will need to be rewritten to create avro instead.


Renaming HVAC topic attributes
------------------------------

There is a proposal to rename many topic items in the HVAC (OSW-2192).
This is probably the time to do it since (as Michael suggested) we will create a new database.

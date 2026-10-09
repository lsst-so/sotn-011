
.. note::

  Add details about the proposal.

Initial Translation
-------------------

Initial translation of topics from xml into avro.
This is probably where we want to enforce the refactoring as well which means, whenever a component interface is translated into avro we will drop the use of arrays.
Will be ok for some CSCs and painful for others.

As an example, ``SALGenerics.xml`` declares a global ``SummaryStates`` enumeration and the ``SALGeneric_logevent_summaryState`` topic.
The topic's ``summaryState`` item is currently declared as a scalar ``long``.
An Avro translation could instead use a named enum:

.. code-block:: json

   {
     "type": "record",
     "name": "SALGeneric_logevent_summaryState",
     "namespace": "lsst.sal",
     "fields": [{
       "name": "summaryState",
       "type": {
         "type": "enum",
         "name": "SummaryStates",
         "namespace": "lsst.sal.enums",
         "symbols": [
           "DisabledState",
           "EnabledState",
           "FaultState",
           "OfflineState",
           "StandbyState"
         ],
         "salCodes": {
             "DisabledState": 1,
             "EnabledState": 2,
             "FaultState": 3,
             "OfflineState": 4,
             "StandbyState": 5
         }
       }
     }]
   }

This is a proposed typed representation, not a wire-compatible conversion: the XML lists the enum names but does not specify their integer assignments.
Those assignments must be verified and mapped before replacing the SAL ``long`` field.

Enumerations
------------

Avro named enums provide a set of symbols, but do not associate symbols with explicit SAL integer codes.
Their binary values are the symbols' zero-based positions in the schema.
This is a poor fit for SAL enums whose numeric codes are part of the existing interface: changing symbol order changes the Avro ordinals, while XSD enumeration order does not define semantic numeric values.

XML enumeration values do not necessarily satisfy Avro's symbol-name rules.
Values that need normalization require a stable mapping back to their XML values; where that cannot be done reliably, a string field may be a better representation.
Extra schema properties such as ``salCodes`` can document a mapping, but standard Avro readers do not interpret them.
See the `Apache Avro specification <https://avro.apache.org/docs/1.12.0/specification/>`_ and the `W3C XML Schema enumeration facet <https://www.w3.org/TR/xmlschema-2/#enumeration>`_.

The conclusion is that Avro enums are too restrictive as SAL's shared enum definition: they discard explicit numeric assignments and constrain the symbols and their ordering.
Move in a different direction by keeping SAL topic fields numeric in Avro and generating Python, C++, and Java enum types from one shared definition containing each symbol and its SAL code.
That definition could be exported from the existing Python enums or maintained in a language-neutral IDL such as Protocol Buffers.
Avro would carry the integer; the generated types would provide names and validation in each language.

The example is based on `SALGenerics.xml <https://github.com/lsst-ts/ts_xml/blob/develop/python/lsst/ts/xml/data/sal_interfaces/SALGenerics.xml>`_ and `SALEventSet.xsd <https://github.com/lsst-ts/ts_xml/blob/develop/python/lsst/ts/xml/data/schema/SALEventSet.xsd>`_.

Protobuf Shared Enums
^^^^^^^^^^^^^^^^^^^^^

The sibling ``xml_enums`` prototype demonstrates a possible alternative: keep the Avro topic field numeric and define each enum's symbols and numeric codes in a Protocol Buffers ``.proto`` file.
The prototype contains per-subsystem ``.proto`` definitions and generated Python, C++, and Java bindings.
The generated types would give each language named constants with the same underlying SAL codes, while Avro continues to carry the integer value.

For example, the current ``ts_xml`` summary-state codes can be represented as:

.. code-block:: protobuf

   syntax = "proto3";
   package lsst.ts.xml.enums;

   enum SummaryState {
     SUMMARY_STATE_UNSPECIFIED = 0;
     SUMMARY_STATE_DISABLED = 1;
     SUMMARY_STATE_ENABLED = 2;
     SUMMARY_STATE_FAULT = 3;
     SUMMARY_STATE_OFFLINE = 4;
     SUMMARY_STATE_STANDBY = 5;
   }

Proto3 requires the first enum value to be zero, so ``UNSPECIFIED`` provides that default while the existing SAL state codes remain unchanged.
These assignments match ``ts_xml/python/lsst/ts/xml/sal_enums.py``.
The standalone ``summarystate.proto`` in the prototype instead assigns the five states values zero through four, so it must be corrected before reuse.

The prototype also shows protobuf aliases using ``allow_alias`` for a deprecated synonym.
Python ``IntFlag`` definitions need separate treatment because a protobuf enum does not provide flag-combination semantics.
The folder contains generated bindings but no visible generation script or build instructions, so a production workflow would need to make regeneration reproducible.

The proposal is to maintain the ``.proto`` definitions as the shared enum source and generate the language bindings from them.
The topic schema remains Avro with numeric enum fields, preserving the SAL codes without making Avro enum ordinals part of the interface.

TW: Will we still have these Python enums in ts_xml?
Or the Python CSC will need to dynamically create and import these enums from the special files (aka. from “export them directly”)?

Common Topic Fields
-------------------

Just a reminder that currently we add a few additional fields to the topics when we translate them from xml to avro.
I think we want to continue doing this as having us duplicate those in the interface is probably a bad idea, which means we will still have to do a translation stage from avro -> avro.

Generics and SAL Subsystems
---------------------------

In addition to the xml with the CSCs interfaces we have a pair of xml files that specify the “Generics” and the “SALSubsystems” interfaces.
The Generics can certainly be converted into avro but SALSubsystems.xml is not a topic interface but metadata for each component.
Will we pick a new format for this file or keep xml? Json is a good option (very transportable) but hard on users.
YAML is another option, more readable but less transportable.

(A quote from a podcast: XML is better than YAML, because there are situations where XML is appropriate, but there’s no situation where YAML is appropriate.

MR Counter argument: See Helm chart configuration which I would hate to do in XML.)

MR: Going to throw my 2 cents at this and say JSON would probably be more appropriate.
Bridges the gap between human and machine readable and does not require third party packages in python to parse.
I don’t think JSON is that hard on users given all the IDE support for the format.
Yes, it does take a little bit to get used to. 

Additional Support on SAL/SalObj
--------------------------------

Add an optional field in the write method on salobj to allow passing “measurement time”.
If not set it will fallback to current behavior, if set it overrides the timestamp. 

Improvements to Interface Definition
------------------------------------

Since we are making major updates to the interface definition, what else can we improve?

One idea I have had for some time (based on a comment made by Petr a long time ago) was to have a way for a people to annotate topics/topics attributes to specify it is used somewhere else.
For example, the pointing component reads data from the ESS.
This means someone working on updating the ESS interface can make changes to a topic that can break things on the pointing.
We could envision a mechanism that users of a topic would annotate the topic attributes so others would be aware that they might impact other components.
Of course, manually doing this process is a bit britle, especially if we don't really have a way to enforce it.


.. note::

  Add details about the proposal.

Initial Translation
-------------------

Initial translation of topics from xml into avro.
This is probably where we want to enforce the refactoring as well which means, whenever a component interface is translated into avro we will drop the use of arrays.
Will be ok for some CSCs and painful for others.

Enumerations
------------

There is a consolidation of the different mechanisms that needs to happen (inline vs global enums).
Then there’s the question of how to make them broadly usable (e.g. beyond Python).
Maybe we can figure out a way to export them directly instead of having to manually curate the Python enums?

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


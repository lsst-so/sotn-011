
.. note::

  Add details about the proposal.

Initial Translation
-------------------

The first step in this process is providing a mechanism to translate the original XML topics into Avro.
At this stage, we will also enforce topic refactoring by replacing arrays with single-element fields.

Packed data can be directly converted from arrays to single elements by including an additional ``timestamp`` field if one is not already present.

Simultaneous measurements, on the other hand, require a slightly more complex process.
Some of these topics represent measurements from different sensors (categorized by location, ID, or other properties), while others correspond to actuators, motors, or similar hardware.

Another issue to consider involves commands that accept arrays to provide setpoints for multiple actuators simultaneously.
For example, the M1M3 ``applyActiveOpticForces`` command allows setting the mirror's entire force profile in a single operation (with an equivalent case existing for M2).
One alternative is splitting these actions into separate "setting" and "apply" phases.
During the setting phase, target values are updated but not applied; verification and physical execution then take place during the apply phase.
However, this approach turns a straightforward, single-phase set/verify/apply operation into a multi-stage process that is more error-prone.
This might be an indication that we should still permit arrays in cases where they make clear practical sense, focusing instead on sanitizing component interfaces to reduce our overall reliance on arrays.
For more information see :ref:`m1m3-case-study` and :ref:`m2-case-study` furthermore.

To support the initial translation phase, we can provide a Python script in ``ts_xml``.
For topics without arrays, the script can automatically generate the equivalent Avro schema directly from the XML.
For packed data, the script can likely emit the correct schema with minimal input.
Simultaneous measurements present a more complex scenario: while the script can generate the correct Avro schema for most cases with minimal input (e.g., specifying the name of the index field), some topics may require manual intervention or a complete rewrite by a domain expert.
We should also consider using an AI agent to assist with the most complex cases, as LLMs can perform the translation effectively when guided by user input.

Enumerations
------------

There is a consolidation of the different mechanisms that needs to happen (inline vs global enums).
Then there’s the question of how to make them broadly usable (e.g. beyond Python).
Maybe we can figure out a way to export them directly instead of having to manually curate the Python enums?

TW: Will we still have these Python enums in ts_xml?
Or the Python CSC will need to dynamically create and import these enums from the special files (aka. from “export them directly”)?

Common Topic Fields
-------------------

Currently, during the XML-to-Avro translation stage, a set of common header and system metadata fields are automatically injected into each topic schema.
These fields include system tracking parameters such as ``salIndex`` (for indexed components), publication/reception timestamps (``private_sndStamp``, ``private_rcvStamp``, ``private_efdStamp``, ``private_kafkaStamp``), sequence numbers (``private_seqNum``), interface revision hashes (``private_revCode``), and publisher origin details (``private_identity``, ``private_origin``).

We propose continuing this practice rather than requiring developers to duplicate these standard fields manually in every individual topic definition.
Maintaining centralized injection ensures consistency across interfaces and reduces boilerplate code.
Consequently, even with topic interfaces defined natively in Avro, a lightweight schema processing pipeline (an Avro-to-Avro transformation step) will remain necessary to inject these common fields prior to publishing schemas to the schema registry.

Generics and SAL Subsystems
---------------------------

In addition to the XML files defining the individual CSC interfaces, two core schema files exist: ``Generics.xml`` and ``SALSubsystems.xml``.

While ``Generics.xml`` defines reusable topic interfaces and can be converted directly into Avro schemas, ``SALSubsystems.xml`` serves a different purpose: it contains metadata describing each component rather than topic interface definitions.
Consequently, converting ``SALSubsystems.xml`` to Avro is not appropriate, requiring a decision on whether to retain XML or transition to a alternative serialization format.

While YAML is often considered for human readability, it presents interoperability challenges and requires external parsing dependencies.
JSON offers a better balance for this metadata:

* **Standard Library Support:** Native JSON parsing is built into Python's standard library (``json``), eliminating third-party package dependencies.
* **Interoperability:** JSON provides seamless machine readability and cross-language support, bridging human readability with robust automated tooling.
* **Modern Tooling:** Modern IDEs offer schema validation, formatting, and syntax highlighting, mitigating usability concerns for manual editing.

Therefore, we propose retaining Avro for all topic definitions (including generics) while converting component metadata in ``SALSubsystems.xml`` to JSON.

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


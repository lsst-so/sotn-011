##########################################################
Refactoring the Rubin Observatory Control System Interface
##########################################################

.. abstract::

   The Vera C. Rubin Observatory Control System Interface was originally designed at the early stages of the project, based on knowledge and technology available at the time.
   Over the years, the system itself underwent major re-architecting and upgrades, including a major upgrade of the middleware from DDS to Kafka.
   However, most of the interface concepts, definitions and technology remained unchanged.
   For example, we still rely on xml files to define the components interfaces, even though the backend of the system have evolved to rely on avro.
   Construction and commissioning tight schedule were the main anchor keeping the interface definitions unchanged.
   With the onset of operations we now have more bandwidth to tackle this issue.
   Here we will go over the details regarding the refactoring of the Rubin Observatory Control System Interface.
   We will discuss the main reasons behind the proposed changes, their benefits and potential drawbacks.
   We will also go over a few challenging use-cases.

.. NOTE: With the exception of the abstract, do not add any content to this file.
   All content should be included in separate files, separated by sections.

Introduction
============

.. include:: introduction.rst

Performance
===========

.. include:: performance.rst

From xml to avro
================

.. include:: xml-to-avro.rst

Use-cases
=========

.. include:: use-cases.rst

Timeline
========

.. include:: timeline.rst

Conclusions
===========

.. include:: conclusions.rst

.. image:: https://img.shields.io/badge/sotn--011-lsst.io-brightgreen.svg
   :target: https://sotn-011.lsst.io
.. image:: https://github.com/lsst-so/sotn-011/workflows/CI/badge.svg
   :target: https://github.com/lsst-so/sotn-011/actions/

##########################################################
Refactoring the Rubin Observatory Control System Interface
##########################################################

SOTN-011
========

The Vera C. Rubin Observatory Control System Interface was originally designed at the early stages of the project, based on knowledge and technology available at the time. Over the years, the system itself underwent major re-architecting and upgrades, including a major upgrade of the middleware from DDS to Kafka. However, most of the interface concepts, definitions and technology remained unchanged. For example, we still rely on xml files to define the components interfaces, even though the backend of the system have evolved to rely on avro. Construction and commissioning tight schedule were the main anchor keeping the interface definitions unchanged. With the onset of operations we now have more bandwidth to tackle this issue. Here we will go over the details regarding the refactoring of the Rubin Observatory Control System Interface. We will discuss the main reasons behind the proposed changes, their benefits and potential drawbacks. We will also go over a few challenging use-cases.

**Links:**

- Publication URL: https://sotn-011.lsst.io
- Alternative editions: https://sotn-011.lsst.io/v
- GitHub repository: https://github.com/lsst-so/sotn-011
- Build system: https://github.com/lsst-so/sotn-011/actions/


Build this technical note
=========================

You can clone this repository and build the technote locally if your system has Python 3.11 or later:

.. code-block:: bash

   git clone https://github.com/lsst-so/sotn-011
   cd sotn-011
   make init
   make html

Repeat the ``make html`` command to rebuild the technote after making changes.
If you need to delete any intermediate files for a clean build, run ``make clean``.

The built technote is located at ``_build/html/index.html``.

Publishing changes to the web
=============================

This technote is published to https://sotn-011.lsst.io whenever you push changes to the ``main`` branch on GitHub.
When you push changes to a another branch, a preview of the technote is published to https://sotn-011.lsst.io/v.

Editing this technical note
===========================

The main content of this technote is in ``index.rst`` (a reStructuredText file).
Metadata and configuration is in the ``technote.toml`` file.
For guidance on creating content and information about specifying metadata and configuration, see the Documenteer documentation: https://documenteer.lsst.io/technotes.

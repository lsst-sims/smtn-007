:tocdepth: 1

.. note::

   **This technote is not yet published.**

Introduction
============

The Simulated Observatory Control System (SOCS) is designed to work with the LSST Scheduler. However, this does not mean that other Schedulers can't be developed and interfaced with SOCS in order to produce simulated surveys. This document details the required interfaces and specifications that will allow another Scheduler to be used with SOCS.

Mandatory Requirements
======================

SAL/DDS Communication
---------------------

This is by far the most stringent requirement for any Scheduler. The LSST Observatory Control System (OCS) communicates with all components via the Software Abstraction Layer (SAL)/Distributed Data Service (DDS) protocol. SOCS also communicates this way to interface with the LSST Scheduler. There is an Interface Control Document (ICD) which details currently defined the SAL/DDS topics for Scheduler communication. The `Scheduler ICD`_ is kept in LSST's DocuShare repository. If you do not have access to DocuShare, you can contact the author for a copy. The specification of the topics is kept with the `SAL code`_ on the ``feature/scheduler_xml_devel`` branch. Setup and building of the topic library can be found in the `SOCS documentation`_. **NOTE**: The SAL/DDS system requires running on a Linux environment. Mac support is in the works.

From Scheduler Topics
~~~~~~~~~~~~~~~~~~~~~

There are currently four topics that the LSST Scheduler sends to SOCS, all of which are required for SOCS to correctly function. It is not completely necessary to have all information within a topic filled. Each topic will now be discussed in detail.

Fields
^^^^^^

All information is currently mandatory due to the LSST Scheduler being responsible for the information. However, the new `Sims Survey Fields code`_ is destined end the need for a Scheduler to send this information as all code bases can benefit from the central resource. For mapping sky brightness and airmass, a field Id is required by SOCS. It does not currently offer HealPix support, however, one can work with the author to determine methodology for commensurate behavior.

Next Target
^^^^^^^^^^^

This topic does not need all of the information filled out, however there are some required attributes. The required ones are targetId, filter request_time, request_mjd, ra, dec, num_exposures, exposure_times, num_proposals and proposal_Ids. The proposal Ids are served by SOCS, so one should receive the proposal configuration topics to get the correct Ids. 

Filter Swap
^^^^^^^^^^^

Since the LSST camera cannot house all filters at a given time, the LSST Scheduler keeps track of the moon phase to determine if a filter swap is necessary. A Scheduler may handle filter swapping in a different manner, but the topic message is required by SOCS even if the need_swap attribute is always False.

Interested Proposal
^^^^^^^^^^^^^^^^^^^

This topic is also required in the sense that it needs to be sent to SOCS after an observation is made, but an empty message may be used. An empty message means that the num_proposals attribute is zero which stops SOCS from recording any information into the survey database.

To Scheduler Topics
~~~~~~~~~~~~~~~~~~~

The topics that all Schedulers must handle are listed in the following sections.
 
Time Handler
^^^^^^^^^^^^

This topic provides the wall clock for the simulation (timestamp and night) as well as downtime information for the observatory. The downtime information is particularly important as SOCS will skip ahead to the next night when downtime is declared. Any Scheduler must be able to handle this situation gracefully.

Observing Site
^^^^^^^^^^^^^^

This configuration topic is mandatory since all Schedulers should not have predefined values for this information. SOCS uses a sanctions package to retrieve the information to cut down on errors related to uncontrolled parameter changes.

Observatory Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^

The observatory model configuration topics, Telescope, Telescope Rotator, Dome, Camera, Slew Optics Loop Corrections and Park, must all be received by any Scheduler and used to configure the mandatory observatpry model. This model will be discussed in a later section.

Proposal Configuration
^^^^^^^^^^^^^^^^^^^^^^

The Area Distribution Proposal and forthcoming Time-Domain Proposal configuration topics must be received by any Scheduler. SOCS is the keeper of the baseline survey parameters and any Scheduler must conform to these as they represent the sanctioned science configuration. Since SOCS is the gatekeeper for survey parameter exploration, any Scheduler must be able to handle variations in the parameters for successful simulation completion.

Seeing
^^^^^^

SOCS uses the Opsim3 seeing information in the topic it broadcasts to the LSST Scheduler. This is currently the only sanctioned seeing information available for use. It is the zenith seeing (arcseconds) at 500 nm.

Observatory State
^^^^^^^^^^^^^^^^^

This topic contains the current observatory state after an observation is made. It is necessary to use the information to update the observatory model in a Scheduler to ensure correct slew time calculations.

EUPS Compatible
---------------

Any new Scheduler code base must conform to the EUPS configuration system used by Data Management (DM). This makes setup and running the overall system easier to manage. A short guide to making an EUPS compatible package can be found in the `EUPS Building`_ section of the DM developer guide. Both the `SOCS code`_ and `Scheduler code`_ conform to this, so they provide a good place to start.

Scheduler Executable
--------------------

SOCS requires that any Scheduler wishing to interface with it must be launchable via a script called ``scheduler.py``. This script should be configured to be in the ``$PATH`` for the Scheduler package. This allows for easy generic execution by SOCS. If a Scheduler is written to use the Python logging system, it must hook into the central logging process launched by SOCS. See the LSST `Scheduler code`_ ``setup.configure_logging`` function for details on how this works. The first three arguments in the ``setup.create_parser`` function shows how SOCS needs to interact with a Scheduler to configure logging. The The SOCS code can be profiled by using a switch passed to the executable. A Scheduler may wish to have profiling run at the same time. If so, it should support the command line argument ``--profile`` for this to occur.

Observatory Model
-----------------

The current `Scheduler code`_ contains the only sanctioned observatory model for the project. It must be used to calculate all slew times for ranking targets. SOCS provides the configuration for the observatory via a DDS/SAL topic that any Scheduler must recieve, convert and pass that to the instantiation of the observatory model. Examples of use can be found in the `Scheduler code`_ repository. Again, all Schedulers **MUST** use this model even if no other code from the LSST Scheduler is used. It is possible that this code may be refactored into a separate repository for easier use.

Sky Brightness Model
--------------------

SOCS and the Scheduler will be moving to the pre-calculated `Sky Brightness code`_ by the time version 1.0 of the combined system comes out. This is a project sanctioned model for sky brightness. This is the model that SOCS will use to determine the sky brightness at the time of observation. Any Scheduler should make use of this model to ensure commensurate sky brightness behavior between the two systems.

Optional Requirements
=====================

SAL/DDS Communication
---------------------

To Scheduler Topics
~~~~~~~~~~~~~~~~~~~

Scheduler Configuration
^^^^^^^^^^^^^^^^^^^^^^^

The Scheduler Driver and Scheduler configuration topics are used by the LSST Scheduler. However, other Scheduler's are not required to use them. If configuration of another Scheduler is required, please work with the SOCS author to ensure proper communication of parameters.

Clouds
^^^^^^

SOCS currently uses the OpSim3 cloud information. A Scheduler may have it's own cloud model and ignore the information coming from SOCS. However, the observation cloud information is taken from the OpSim3 values. If a Scheduler author wishes to have commensurate behavior, please work with the SOCS author to design a suitable mechanism for information gathering.

Observation
^^^^^^^^^^^

While this is an important topic, it is possible that a given Scheduler may not use the information stored within the observation topic to track progress or for other bookkeeping. That Scheduler may have alternate methodologies for handling this information. Therefore, the topic is not required for use, but will be sent regardless.

.. _SOCS code: https://github.com/lsst-sims/sims_ocs
.. _SOCS documentation: https://lsst-sims.github.io/sims_ocs
.. _Scheduler ICD: https://docushare.lsstcorp.org/docushare/dsweb/Services/Document-18105
.. _SAL code: https://github.com/lsst-ts/ts_sal
.. _Scheduler code: https://github.com/lsst-ts/ts_scheduler 
.. _EUPS Building: https://developer.lsst.io/build-ci/eups_tutorial.html#building-and-using-a-simple-eups-product
.. _Sky Brightness code: https://github.com/lsst/sims_skybrightness_pre
.. _Sims Survey Fields code: https://github.com/lsst/sims_survey_fields
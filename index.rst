..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

Background
==========
   
The EFD (Engineering Facilities Database) is a system developed by LSST Telescope and Site that consists of an SQL database as well as the Large File Annex, which is a repository of binary data such as images from the all-sky camera. In this document, we will use EFD to refer to the database part, and the EFD's Large File Annex shall be explicitly referenced as such.

The EFD is populated from information published to the service abstraction layer (SAL); that includes telemetry for all sensors and devices on the summit, as well as other information that Telescope and Site need to store, such as the (planned) night observer's log. 

The EFD contains data that is of interest to users of DM systems and services, including the commmissioning team, the science validation team and the science pipelines team. This data is most useful when co-analysed or co-monitored with data derived by the science pipelines. This unofficial technote discusses an alternative proposal to the baselined plan for servicing these users. 

Exposing EFD data to DM tooling
===============================

There are a number of motivators for exposing EFD data to DM tools and services:

* Some investigations will require correlating facility and environmental events with each other, such as correlating wind speed and direction, the position of the louvres in the dome and the PSF derived from observations. These usecases are particularly prevalent during the commissioning and verification phases of the facility. In this class of usecase, telemetry-derived, data-derived and human-originating information is co-analysed to characterise and diagnose issues with the facility. These can go either way: using the EFD data as the primary data discovery index (eg show all observations at low eastern elevations on windy nights) or the reverse (for these observations, plot the focal plane temperature sensor data). 

* DM is building powerful tooling to support both ad-hoc investigation (chiefly the Notebook Aspect of the Science Platform); these significantly exceed any planned capabilities from Telescope and Site for analysing EFD data; therefore prompt exposure of this data to the Science Platform will be of direct benefit to EFD data consumers in other subsystems. These explorations include freeform investigations into EFD data ("plot everything against everything").

* Additionally, DM (SQuaRE) has built a framework for metric curation and alert monitoring (lsst.verify, SQuaSH and a planned trend excursion alert harness) that could be well suited for integrating certain key facility metrics.

* Aggregations of telemetry data are also of interest to night-reporting and characterisation-reporting tooling.


Baselined Design
================

The raw EFD is optimised for the high frequency writes that are expected from a telemetry database. The intent has been to have a version XXXXX

The baseline design involves an ETL (extract-transform-load) process from the "raw" EFD to the retransformed EFD. In the original design the periodicity of EFD data being available in the retransformed EFD was set to once every 24 hours. Shorter periods such as one hour have been discussed, with the possibility going down to 5 minutes having been discussed, but not demonstrated as feasible with this architecture. 

In this design, any calibration or aggregation is done during the ETL process. 


Concerns about the Baseline
===========================

There are a number of concerns about the baselined design.

* The major one is the long latency in the availability of the data. We propose that the latency should be equal to (if not less than) the time it takes for an observation or a transient event to be available through the DM Science Platform to users. It is absurd to have a gigapixel sized image available to a Science Platform user but not the wind-speed during its observation. Moreover the strong interest in the XXXX

* The second one is deployment cadence and interface cleanliness. In the baseline design, a desired restructuring of either the raw EFD or the DM-EFD schema involves three systems (the raw EFD, the transformed EFD and the ETL process). One of those (the raw EFD) is likely to be strictly changed controled, wheras DM data services are expected to evolve more frequently on the face of user needs. 

* The third one is that if the availability of EFD data is poor through the ETL-EFD, there will be pressure from the commissinion team to expose the raw-EFD to the Science Platform. We have strong architectural and maintainance concerns over such an event, and moreover it is not resourced. [KSK] Specifically, maintaining multiple interfaces, raw-EFD and ETL-EFD, has an impact on resource allocations that has not been planned.
  
Proposed modification
=====================

We propose that we abandon the ETL process in favour of a direct tap off the Base EFD writers and that we introduce a solution such as Kafka (https://kafka.apache.org/) to handle streaming, caching and aggregation to a DM-specific telemetry database, which we call DM-EFD. This solution can meet the proposed latency requirements and has a weaker coupling between the highly controlled EFD schema and the more rapidly evolving DM services.

Additionally we propose that DM-EFD hold only telemetry data, and that data originating from human comments (eg shiftlog and data quality remarks) be segregated in separate tooling and databases, in order to optimize user-friendly interfaces (eg. Slack) and multi-platform broadcasts (eg. a message goes both in a database and echoed on Slack). 

Both of these would require LCRs and possibly the reallocation of resources.

A straw-man architecture for these modifications is shown in the diagram below

.. figure:: /_static/dm-efd-take2.png
        :name: fig-arch




Design-neutral Requirements
===========================

Rehardless of whether the ETL or new proposed architecture is adopted, the eventual architecture needs to show how it can meet the following requirements. 


Availability of the DM-EFD capabilities
----------------------------------------

If, as anticipated, DM tooling is the primary of interface to EFD data for anyone beyond hardware-level engineers, availability of those services will be important to operational staff in Chile and the US, as well as to science users. It is therefore a requirement that the entire architecture is structured so that sandbox deployments, rolling upgrades and carefully coordinated downtime are the norm for routine operations. 

Interfaces
----------

Data should be available via TAP/ADQL services as other data sources available to the Science Platform.

The interface to the Science Platform should be deployment- and time-invariant: the same notebook accessing EFD data should run without modifications on the day in Chile and a month later at the LDF. 

Aggregation
-----------

There will be a mechanism to aggregate the event streams individually.  There will be at least two kinds of aggreagation:

1. Moving average -- This is for values that are expected to vary smoothly: e.g. temperature.  As well as the average value, the variance, min and max for the window should also be stored.
2. State change -- Values that have discrete stages should be aggregated differently than continuous attributes.  This could be done by just storing the events where a state has changed, however, this may make it hard to query for the state at a particular time because you have to look up the record previous to a particular time stamp.  Another option would be to store the state values on some sampling rate.

Latency
-------

Latency should be addressed in two parts:

1. Persistence latency -- This is the latency between an even being published on the DDS to that event showing up as an aggregated quantity in the DM EFD.  This latency should be equal to or less than the time to take and reduce a single raft of data on a parallel reduction system.  This puts an upper bound on the sampling rate for the aggregated event streams.
2. Query latency -- Doing a strict time span query should be of order 1 sec.  More complicated queries, queries involving joins, will have higher latency and should be addressed on a case by case basis.

Redundancy
----------

The storage system should be sized to hold the aggregated event streams for the duration of commissioning.  It should be redundant, or backed up so that the risk of data loss is acceptably low.

Other
-----

Units should be SI units, and the time stamps should be in UTC.

Example use cases
=================

* Correlate wind speed and direction -- Select the values of wind speed in m/s and direction in degrees from north in a window of time specified in UTC.
* Look for extreme temperature gradients for images with bad seeing -- Select start and stop times for exposures with seeing > 1.2 arcsec.  Select the dome temperature at the start and stop times for each of the exposures.  Plot delta T vs seeing.
* Generic correlation -- Select all relevent values from the EFD for all images taken in a time window.  Associate a typical value with each exposure.  Plot everything against everything.
* Search for data with possible excursions -- We see evidence that when the dome opening is pointing east we have image quality issues.  In order to get a large sample to do the debugging on find all entries in the DM EFD where the dome opening is set to be pointing east.  Next select all exposures where the start/stop times overlap those entries.

Large File Annex
================

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :encoding: latex+latin
..    :style: lsst_aa

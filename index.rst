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

Scope
=====

This document discusses how data from the Engineering Facilities Database and the Large File Annex can be most effectively made available to Science Platform users. 


Background
==========
   
The EFD (Engineering Facilities Database) is a system developed by LSST Telescope and Site that consists of an SQL database as well as the Large File Annex, which is a repository of binary data such as images from the all-sky camera.

.. note::
  
  In this document, we will use EFD to refer to the D (database) part, and the Large File Annex that is part of the EFD system shall be explicitly referenced as such.

The EFD is populated from information published to the service abstraction layer (SAL); that includes telemetry for all sensors and devices on the summit, as well as other information that Telescope and Site need to store, such as the (planned) night observer's log. 

The EFD contains data that is of interest to users of DM systems and services, including the commmissioning team, the science validation team and the science pipelines team. This data is most useful when co-analysed or co-monitored with data derived by the science pipelines. This informal technote discusses an alternative proposal to the baselined plan for servicing these users. 

Example use cases
=================

The Science Platform is intended to support free-form troubleshooting for the Commissioning and Science Validation teams, so obtaining an exhaustive list of usecases is difficult. As long as data from the EFD is exposed in good time and in good form to the Science Platform, we expect to be able to service a large portion of both the anticipated and unanticipated use cases.

Following are some example investigations we expect to be representative of core activities users of the Science
Platform will undertake: 

* Correlate wind speed and direction -- Select the values of wind speed in m/s and direction in degrees from north in a window of time specified in UTC.
* Look for extreme temperature gradients for images with bad seeing -- Select start and stop times for exposures with seeing > 1.2 arcsec.  Select the dome temperature at the start and stop times for each of the exposures.  Plot delta T vs seeing.
* Generic correlation -- Select all relevent values from the EFD for all images taken in a time window.  Associate a typical value with each exposure.  Plot everything against everything.
* Search for data with possible excursions -- We see evidence that when the dome opening is pointing east we have image quality issues.  In order to get a large sample to do the debugging on find all entries in the DM EFD where the dome opening is set to be pointing east.  Next select all exposures where the start/stop times overlap those entries.
* Real time trend andalysis and diagnostics -- The commissioning scientist wants to monitor several different telemetry quantities on a custom dashboard intended to reveal, in real time, correlations and trends of various related telemetry streams.
The health dashboard may have some of these quantities, but it will not be configurable the same way a custom dashboard could be.
It also provides better accessibility since the summit health check dashboard is not expected to be accessible from outside the dome.
A concrete example of this usage is a scenario where the commissioning scientist wants to modify the set point of the focalplane temperature controller.
They then want to trigger wavefront observations at intervals as the system returns to equilibrium to monitor how the system responds to such stimuli.

Exposing EFD data to DM tooling
===============================

There are a number of motivators for exposing EFD data to DM tools and services:

* Some investigations will require correlating facility and environmental events with each other, such as correlating wind speed and direction, the position of the louvres in the dome and the PSF derived from observations. These usecases are particularly prevalent during the commissioning and verification phases of the facility. In this class of usecase, telemetry-derived, data-derived and human-originating information is co-analysed to characterise and diagnose issues with the facility. These can go either way: using the EFD data as the primary data discovery index (eg show all observations at low eastern elevations on windy nights) or the reverse (for these observations, plot the focal plane temperature sensor data). 

* DM is building powerful tooling to support both ad-hoc investigation (chiefly the Notebook Aspect of the Science Platform); these significantly exceed any planned capabilities from Telescope and Site for analysing EFD data; therefore prompt exposure of this data to the Science Platform will be of direct benefit to EFD data consumers in other subsystems. These explorations include freeform investigations into EFD data ("plot everything against everything").

* Additionally, DM (SQuaRE) has built a framework for metric curation and alert monitoring (lsst.verify, SQuaSH and a planned trend excursion alert harness) that could be well suited for integrating certain key facility metrics.

* Aggregations of telemetry data are also of interest to night-reporting and characterisation-reporting tooling.


Baselined Design
================

The raw EFD is optimised for the high frequency writes that are expected from a telemetry database. The intent has been to have a separate database, derived from data in the raw EFD, in the LDF for use by the Data Release Production pipelines.

The baseline design involves an ETL (extract-transform-load) process from the "raw" EFD to the retransformed EFD. In the original design the periodicity of EFD data being available in the retransformed EFD was set to once every 24 hours. Shorter periods such as one hour have been discussed, with the possibility going down to 5 minutes having been discussed, but not demonstrated as feasible with this architecture. 

In this design, any calibration or aggregation is done during the ETL process. 


Concerns about the Baseline
===========================

There are a number of concerns about the baselined design.

* The major one is the long latency in the availability of the data. We propose that the latency should be equal to (if not less than) the time it takes for an observation or a transient event to be available through the DM Science Platform to users. It would be unfortunate to be in the situation where a gigapixel sized image is available to a Science Platform user but not the wind-speed during its observation.

* The second one is deployment cadence and interface cleanliness. In the baseline design, a desired restructuring of either the raw EFD or the DM-EFD schema involves three systems (the raw EFD, the transformed EFD and the ETL process). One of those (the raw EFD) is likely to be strictly changed controled, wheras DM data services are expected to evolve more frequently on the face of user needs. 

* The third one is that if the availability of EFD data is poor through the ETL-EFD, there will be pressure from the commissinion team to expose the raw-EFD to the Science Platform. We have strong architectural and maintainance concerns over such an emerging requirement, and moreover it is not resourced. Specifically, maintaining multiple interfaces, raw-EFD and ETL-EFD, has an impact on resource allocations that has not been planned.
  
Proposed modification
=====================

Rather than going through an ETL process, we propose a solution that uses a direct tap off the Base EFD writers. Such a solution would handle the streaming, caching and aggregation to a DM-specific telemetry database, which we call the DM-EFD. This solution can meet the proposed latency requirements and has a weaker coupling between the highly controlled EFD schema and the more rapidly evolving DM services, instead of a schema-schema transform. 

A technology in use elsewhere in the project (for alert distribution) is Kafka (https://kafka.apache.org/). Kafka can handle streaming, caching and aggregation out of the box, so may prove to be a very good fit for the system proposed here. Whether aggregation is handled before publishing to a Kafka-like system or within the system itself is an open question as benchmarks for publishing streams of the richness expectewd from the SAL have not yet been carried out.

Additionally we propose that DM-EFD hold only telemetry data and events, and that data originating from human comments (eg shiftlog and data quality remarks) be segregated in separate tooling and databases, in order to optimize user-friendly interfaces (eg. Slack) and multi-platform broadcasts (eg. a message goes both in a database and echoed on Slack). 

Both of these would require LCRs and possibly the reallocation of resources.

A straw-man architecture for these modifications is shown in the diagram below

.. figure:: /_static/dm-efd-take2.png
        :name: fig-arch

In the EFD design there is a SAL client that monitors the DDS bus and uses writers to insert telemetry values into the EFD, write them in logs etc. It is a lightweight change to add a writer to publish these values to Kafka. Kafka can both deal with caching and connection management, as well as aggregation. 


Event and Command Streams
=========================

As well as the Telementry stream, the EFD captures Event Streams and Command Streams. Although these streams are of potential interest to the Science Platform users for troubleshooting purposes, they are analogous to log messages - informational rather than quantitative. Therefore we propose that Event and Command streams are treated as Telemetry insofar that they are forwarded by Kafka to be be stored in the DM-EFD for querying, but there is no aggegation necessary. 


Large File Annex
================

The Large File Annex is a store of non-scalar auxillary data, from
images, to FITS cubes and PDF documents. When data from an auxilary
source such as the all-sky camera has been stored in the Large File
Annex, its avaibility is broadcast on the Large File Annex
Announcement Even Stream.

By volume, most of the information in the LFA is of no interest to Science Platform users, nor is it in a form that is tractable for python-level exploitation. For example, the LFA contains reports in the form of Excel spreadsheets; a Science Platform user is likely to create reports from the data directly, rather than interact with the derived documents.

Data of interest in the LFA originates from:

* The All-Sky Camera

* Guider images

* Composite Wavefrong Images

* Laser (KSK: I'm unclear exactly what these laser data are.  Robert specifically said he didn't necessarily need the positioning laser data)

* Flatfield screen monochrometer

* Sky-spectrum monitor (if/when built)

Like the data from the Auxillary Telescope, users want to interact with the LFA data through the butler, an appropriate dataset type having been define. Moreover users require these data with very low latencies as it is likely that they need it in order to make on-the-fly adjustments to systems during commissioning.

We therefore propose that the LFA Announcement Stream is monitored by the DM-CS and when data from these enumerated sources is made available, that it be injected into the data backbone, from whence it will be treated like data (and not telemetry) by upstream services.


Design-neutral Requirements
===========================

Rehardless of whether the ETL or new proposed architecture is adopted, the eventual architecture needs to show how it can meet satisfy the following requirments and use cases.  


Availability of the DM-EFD capabilities
----------------------------------------

If, as anticipated, DM tooling is the primary of interface to EFD data for anyone beyond hardware-level engineers, availability of those services will be important to operational staff in Chile and the US, as well as to science users. It is therefore a requirement that the entire architecture is structured so that sandbox deployments, rolling upgrades and carefully coordinated downtime are the norm for routine operations. 

Interfaces
----------

Data should be available via TAP/ADQL services as other data sources available to the Science Platform.

The interface to the Science Platform should be deployment- and time-invariant: the same notebook accessing EFD data should run without modifications on the day in Chile and a month later at the LDF.

A syntactic sugar to make access to EFD data more pythonic from the notebook (and to shield the user from schema implementation details) has been requested. Here is an example of how a notebook user could obtain statistics on the M1/M3 temperature sensors::

  import numpy as np
  import lsst.efd as efd
  ...

  # Get the temperatures in one go 
  envtemp = efd.get("m1m3.actuators.envtemp")
  stdev = np.std(envtemp)
  mean = np.mean(envtemp)
  print(f"temperature = {mean} +/- {stdev} K")



Aggregation
-----------

The purpose of aggregation is both to reduce volume on high-frequency telemetry data and to increase the signal-to-noise of busy telemetry. Science Platform users are generally interested in events at the same order of cadence as a camera exposure; therefore we propose that all telemetry data sampled with a frequency higher than 1Hz is (1) sampled at 1Hz and (2) aggregated to 1Hz using these generic statistics:

* Max

* Min

* Mean

* Median

* Standard Deviation

For command streams, no aggregation should be done.

For event streams we propose that using Kafka we sum repeats of the
same messages within the 1Hz window (eg if the M1M3 subsystem issues a
limitError even 100 times in the last second, a repeat counter of 100
is stored with the event).


Latency
-------

Latency should be addressed in two parts:

1. Persistence latency -- This is the latency between an even being published on the DDS to that event showing up as an aggregated quantity in the DM EFD.  This latency should be equal to or less than the time to take and reduce a single raft of data on a parallel reduction system.  This puts an upper bound on the sampling rate for the aggregated event streams. For Auxillary data, lower latencies are required; for example CBP data has been requested to be available at 1-second scale latencies. 

2. Query latency -- Doing a strict time span query should be of order 1 sec.  More complicated queries, queries involving joins, will have higher latency and should be addressed on a case by case basis.

Redundancy
----------

DM-EFD should be sized to hold the aggregated event streams from commissioning to the end if operations.  It should be redundant, or backed up so that the risk of data loss is acceptably low, even if the EFD system is backed some other way into cold storage. 

Other
-----

A notebook examining data should be deployment invariant within LSST operations; i.e. the same notebook should work in a Science Platform deployment at the LDF and one at the Base. 

Units should be SI units, and the time stamps should be in UTC.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :encoding: latex+latin
..    :style: lsst_aa

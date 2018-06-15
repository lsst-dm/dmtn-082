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

   The EFD is a powerful tool for correlating pipelines behavior with the state of the observatory.  Since it contains the logging information published to the service abstraction layer (SAL) for all sensors and devices on the summit, it is a one stop shop for looking up the state of the observatory at a given moment.  The expectation is that the science pipelines validation, science verification, and commissioning teams will all need, at one time or another, to get information like this for measuring the sensitivity of various parts of the system on observatory state: e.g. temperature, dome orientation, wind speed and direction, gravity vector.  This leads to the further expectation that the various DM and commissioning teams will want to work with a version of the EFD that is accessed like a traditional relational database, possibly with linkages between individual exposures and specific pieces of observatory state.  This implies the need for another version of the EFD (called the DM-EFD here) that has had transforms applied to the raw EFD that make it more immediately applicable to questions the DM team will want to ask.
   
High level requirements
=======================

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

Interface
---------

All event streams should be in calibrated units where applicable.  An example is that if the exhaust fans report voltage, the stored value should be in units of RPM for the fan blades.

The interface should be a standard interface supported by third party python libraries.  The obvious choice would be a SQL interface possibly implemented with SQLAlchemy.

Units should be SI units, and the time stamps should be in UTC.

Example use cases
=================

* Correlate wind speed and direction -- Select the values of wind speed in m/s and direction in degrees from north in a window of time specified in UTC.
* Look for extreme temperature gradients for images with bad seeing -- Select start and stop times for exposures with seeing > 1.2 arcsec.  Select the dome temperature at the start and stop times for each of the exposures.  Plot delta T vs seeing.
* Generic correlation -- Select all relevent values from the EFD for all images taken in a time window.  Associate a typical value with each exposure.  Plot everything against everything.
* Search for data with possible excursions -- We see evidence that when the dome opening is pointing east we have image quality issues.  In order to get a large sample to do the debugging on find all entries in the DM EFD where the dome opening is set to be pointing east.  Next select all exposures where the start/stop times overlap those entries.

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :encoding: latex+latin
..    :style: lsst_aa

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

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :encoding: latex+latin
..    :style: lsst_aa

.. image:: https://img.shields.io/badge/dmtn--082-lsst.io-brightgreen.svg
   :target: https://dmtn-082.lsst.io
.. image:: https://travis-ci.org/lsst-dm/dmtn-082.svg
   :target: https://travis-ci.org/lsst-dm/dmtn-082
..
  Uncomment this section and modify the DOI strings to include a Zenodo DOI badge in the README
  .. image:: https://zenodo.org/badge/doi/10.5281/zenodo.#####.svg
     :target: http://dx.doi.org/10.5281/zenodo.#####

#######################################################################################
High level scope for the DM facing version of the engineering facilities database (EFD)
#######################################################################################

DMTN-082
========

The EFD is a powerful tool for correlating pipelines behavior with the state of the observatory.  Since it contains the logging information published to the service abstraction layer (SAL) for all sensors and devices on the summit, it is a one stop shop for looking up the state of the observatory at a given moment.  The expectation is that the science pipelines validation, science verification, and commissioning teams will all need, at one time or another, to get information like this for measuring the sensitivity of various parts of the system on observatory state: e.g. temperature, dome orientation, wind speed and direction, gravity vector.  This leads to the further expectation that the various DM and commissioning teams will want to work with a version of the EFD that is accessed like a traditional relational database, possibly with linkages between individual exposures and specific pieces of observatory state.  This implies the need for another version of the EFD (called the DM-EFD here) that has had transforms applied to the raw EFD that make it more immediately applicable to questions the DM team will want to ask.

**Links:**

- Publication URL: https://dmtn-082.lsst.io
- Alternative editions: https://dmtn-082.lsst.io/v
- GitHub repository: https://github.com/lsst-dm/dmtn-082
- Build system: https://travis-ci.org/lsst-dm/dmtn-082


Build this technical note
=========================

You can clone this repository and build the technote locally with `Sphinx`_:

.. code-block:: bash

   git clone https://github.com/lsst-dm/dmtn-082
   cd dmtn-082
   pip install -r requirements.txt
   make html

.. note::

   In a Conda_ environment, ``pip install -r requirements.txt`` doesn't work as expected.
   Instead, ``pip`` install the packages listed in ``requirements.txt`` individually.

The built technote is located at ``_build/html/index.html``.

Editing this technical note
===========================

You can edit the ``index.rst`` file, which is a reStructuredText document.
The `DM reStructuredText Style Guide`_ is a good resource for how we write reStructuredText.

Remember that images and other types of assets should be stored in the ``_static/`` directory of this repository.
See ``_static/README.rst`` for more information.

The published technote at https://dmtn-082.lsst.io will be automatically rebuilt whenever you push your changes to the ``master`` branch on `GitHub <https://github.com/lsst-dm/dmtn-082>`_.

Updating metadata
=================

This technote's metadata is maintained in ``metadata.yaml``.
In this metadata you can edit the technote's title, authors, publication date, etc..
``metadata.yaml`` is self-documenting with inline comments.

Using the bibliographies
========================

The bibliography files in ``lsstbib/`` are copies from `lsst-texmf`_.
You can update them to the current `lsst-texmf`_ versions with::

   make refresh-bib

Add new bibliography items to the ``local.bib`` file in the root directory (and later add them to `lsst-texmf`_).

****

Copyright 2018 AURA/LSST

This work is licensed under the Creative Commons Attribution 4.0 International License. To view a copy of this license, visit http://creativecommons.org/licenses/by/4.0/.

.. _Sphinx: http://sphinx-doc.org
.. _DM reStructuredText Style Guide: https://developer.lsst.io/docs/rst_styleguide.html
.. _this repo: ./index.rst
.. _Conda: http://conda.pydata.org/docs/
.. _lsst-texmf: https://lsst-texmf.lsst.io
